---
categories:
  - 기타
tags:
  - db
  - typeorm
  - mysql
date: 2022-07-07 22:41:14
title: TypeORM에서 save를 사용할 때 update되는 기준은 뭘까?
updated:
---


## 1. 계기

지금 다니고 있는 회사에서는 NestJS와 TypeORM을 사용하고 있습니다.
TypeORM을 사용하여 DB에 row를 생성하기 위해서는 `insert`나 `save`를 사용하게 됩니다.

그런데 `save`의 설명을 보면 이렇게 나와있습니다.

{% blockquote typeorm.io https://typeorm.io/entity-manager-api entity-manager-api %}
	`save` - Saves a given entity or array of entities. If the entity already exists in the database, then it's updated. If the entity does not exist in the database yet, it's inserted.
{% endblockquote %}

그런데 문득 궁금증이 생겼습니다. 설명만 보면 마치 `upsert`처럼 동작할 것 같은데 정말로 그럴까요?
또, 여기서 entity가 DB에 **있다**의 기준이 뭘까요?

## 2. Upsert 테스트

`upsert`처럼 동작한다고 가정하고 테스트를 해보도록 하겠습니다.
일반적으로 MySQL에서 `upsert`를 사용한다고 하면 unique가 적용된 컬럼이 있어야겠죠. 아래와 같이 테이블을 만들었습니다.

{% codeblock test 테이블 스키마 lang:SQL  %}
	CREATE TABLE `test` (
		`idx` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
		`name` VARCHAR(50) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
		`num` INT(10) NULL DEFAULT NULL,
		PRIMARY KEY (`idx`) USING BTREE,
		UNIQUE INDEX `uq` (`name`) USING BTREE
	)
	COLLATE='utf8_general_ci'
	ENGINE=InnoDB
	;

{% endcodeblock %}

간단한 NestJS API를 만들고 호출하면서 테스트해보겠습니다.

{% codeblock Test Entity lang:TypeScript %}
	@Entity()
	export class Test {
		@PrimaryGeneratedColumn()
		public idx: number;

		@Column({ length: 50 })
		public name: string;

		@Column()
		public num: number;
	}
{% endcodeblock %}

name에는 `name1`, num에는 `1` 값이 들어가도록 호출했습니다.

{% asset_img 1.JPG %}

{% codeblock 실행된 쿼리 lang:SQL %}
	INSERT INTO `test`(`idx`, `name`, `num`) VALUES (DEFAULT, ?, ?) -- PARAMETERS: ["name1",1]
{% endcodeblock %}

이 상태에서 unique인 name이 같게 `name1`로, num은 `2`로 호출하도록 하겠습니다.
만약 `upsert`로 동작한다면 `ON DUPLICATE KEY UPDATE`가 실행되면서 num이 2로 변경되어야 합니다.

하지만 실제 실행된 쿼리는 그냥 `insert`문입니다.

{% codeblock 실행된 쿼리 lang:SQL %}
	INSERT INTO `test`(`idx`, `name`, `num`) VALUES (DEFAULT, ?, ?) -- PARAMETERS: ["name1",2]
{% endcodeblock %}

따라서 아래와 같이 에러가 발생합니다.

{% asset_img 2.JPG %}

## 3. PK 테스트

`upsert`로 동작하지 않는다는 것은 알았습니다. 그러면 save 호출시 데이터가 **있다**고 판단해서 `update`를 하는 경우는 언제 발생할까요?
제일 만만한 것은 PK이니 PK로 가정하고 테스트를 해보겠습니다.

현재 들어간 데이터의 PK는 8입니다. 이 상태에서 entity에 PK를 세팅하고 save를 하면 어떻게 될까요?
즉, 아래와 같은 데이터 저장을 시도해보겠습니다.

{% codeblock lang:JSON %}
	{
		idx: 8,
		name: 'name1',
		num: 2
	}
{% endcodeblock %}

그러면 아래와 같은 쿼리가 실행됩니다.

{% codeblock 실행된 쿼리 lang:SQL %}
	SELECT `Test`.`idx` AS `Test_idx`, `Test`.`name` AS `Test_name`, `Test`.`num` AS `Test_num` FROM `test` `Test` WHERE `Test`.`idx` IN (?) -- PARAMETERS: [8]
	START TRANSACTION
	UPDATE `test` SET `num` = ? WHERE `idx` IN (?) -- PARAMETERS: [2,8]
	COMMIT
{% endcodeblock %}

이번에는 정말로 `update`가 실행되면서 num 값이 변경되었습니다.

{% asset_img 3.JPG %}

entity에 PK 값이 있으면 해당 값으로 테이블을 검색해 row가 존재하면 `update`, 존재하지 않으면 `insert`를 하게 되는 것이죠.
(이 동작은 테이블에 unique가 없어도 동일하게 돌아갑니다.)

## 4. PK가 없다면?

드물겠지만 만약에 PK가 없는 테이블이면 어떻게 될까요?

이 경우는 사실 존재할 수 가 없습니다. Test entity에 `@PrimaryGeneratedColumn()` 데코레이터를 떼는 순간 아래와 같은 에러가 발생합니다.

{% asset_img 4.JPG %}

## 5. 결론

- TypeORM에서 save를 사용할 때 데이터의 존재 유무를 판단하는 것은 PK 값이다.
- `upsert`는 save를 사용하는 것이 아니라 다른 방식을 써야 한다.
- TypeORM entity는 반드시 PK가 있어야 한다.