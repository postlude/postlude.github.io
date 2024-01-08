---
categories:
  - DB
tags:
  - mongodb
date: 2021-04-27 00:00:01
title: Mongoose alias 사용해보기
updated:
---

# 0. 계기

저희 회사에서는 MariaDB와 MongoDB를 사용 중입니다.

그 중 MongoDB는 제가 들어오기도 전 아주 초창기부터 사용 중이다보니 Collection의 몇몇 필드의 네이밍이 현재 사용하는 네이밍과 맞지 않는 것들이 있었습니다.
이미 데이터가 굉장히 많이 쌓여있는 상태라 명령어를 이용해 필드명을 바꾸는 것은 부담이 되었습니다.

그러던 중 Mongoose의 alias를 알게 되었고 이에 대해 정리해보고자 합니다.

# 1. alias란?

우선 공식 문서에 적힌 내용은 다음과 같습니다.

{% blockquote Mongoose Docs https://mongoosejs.com/docs/guide.html#aliases aliases %}
    Aliases are a particular type of virtual where the getter and setter seamlessly get and set another property. This is handy for saving network bandwidth, so you can convert a short property name stored in the database into a longer name for code readability.
{% endblockquote %}

간단하게 말하자면 virtual의 특별한 형태라고 보면 됩니다.

virtual은 MongoDB에 실제로 저장되지는 않지만 마치 존재하는 값인 것처럼 사용할 수 있는 필드입니다.
({% link 참고 https://mongoosejs.com/docs/guide.html#virtuals %})

이 alias를 이용해 필드의 이름을 제가 원하는 다른 이름으로 바꿔서 사용할 수 있습니다.

# 2. 적용

우선 예시를 적용하기 위해 간단한 node 서버를 만들었습니다.
MongoDB의 testdb에 test라는 collection을 사용하는 코드입니다.

{% codeblock app.js lang:JavaScript %}
    const express = require('express');
    const morgan = require('morgan');
    const mongoose = require('mongoose');
    const Test = require('./model/test')

    const app = express();
    const port = 3000;

    app.use(express.json());
    app.use(morgan('dev'));

    app.get('/', async (req, res) => {
        try {
            const result = await Test.findOne(
                {
                    str: 'a'
                }
            );
            res.send(result);
        } catch (err) {
            console.error(err);
            res.sendStatus(500);
        }
    });

    app.post('/', async (req, res) => {
        try {
            const test = new Test({
                str: 'a',
                num: 1
            });
            await test.save();
            res.sendStatus(200);
        } catch (err) {
            console.error(err);
            res.sendStatus(500);
        }
    });

    app.listen(port, async () => {
        console.log('==================== [NODE SAMPLE] ====================');
        console.log(`- PORT : ${port}`);
        await mongoose.connect('mongodb://localhost:27017/testdb', {
            user: 'postlude',
            pass: 'postlude',
            useNewUrlParser: true, // to fix deprecation warning
            useUnifiedTopology: true // to fix deprecation warning
        });
        console.log('========================================================');
    });
{% endcodeblock %}

{% codeblock test.js lang:JavaScript %}
    const mongoose = require('mongoose');
    const { Schema } = mongoose;
    const modelNm = 'Test';

    const TestSchema = new Schema({
        str: {
            type: String
        },
        num: {
            type: Number
        }
    }, {
        collection: 'test'
    });

    module.exports = mongoose.model(modelNm, TestSchema);
{% endcodeblock %}

이 상태에서 Test 모델의 각 필드에 alias를 적용하면 다음과 같습니다.

{% codeblock test.js lang:JavaScript %}
    const mongoose = require('mongoose');
    const { Schema } = mongoose;
    const modelNm = 'Test';

    const TestSchema = new Schema({
        str: {
            type: String,
            alias: 'txt'
        },
        num: {
            type: Number,
            alias: 'int'
        }
    }, {
        collection: 'test'
    });

    module.exports = mongoose.model(modelNm, TestSchema);
{% endcodeblock %}

적용은 이게 전부입니다. 하지만 이렇게만 하면 alias를 이용해 조회하거나 alias로 적용된 이름으로 API 리턴 값을 전달할수는 없습니다.

아래와 같이 코드 상에서 접근해 사용하는 것만 가능합니다.

{% codeblock app.js lang:JavaScript %}
    const result = await Test.findOne(
        {
            str: 'a'
        }
    );
    console.log(result.int); // 1

// ------------------------------------

    const test = new Test({
        str: 'a',
        num: 1
    });
    console.log(test.txt); // a
{% endcodeblock %}

# 3. alias로 조회하기

alias로 find를 하기 위해서는 모델에 내장된 `translateAliases` 함수를 이용하면 됩니다.

{% codeblock app.js lang:JavaScript %}
    const result = await Test.findOne(
        Test.translateAliases({
            txt: 'a'
        })
    );
{% endcodeblock %}

위와 같이 find 조건에 해당하는 객체를 `translateAliases` 함수로 한 번 감싸서 사용하면 동일한 결과 값을 얻을 수 있습니다.

# 4. alias를 리턴하기

alias가 리턴에 포함되지 않는 이유는 Mongoose 문서의 virtual 부분을 보면 알 수 있습니다.

{% blockquote Mongoose Docs https://mongoosejs.com/docs/guide.html#virtuals %}
    If you use toJSON() or toObject() mongoose will not include virtuals by default. This includes the output of calling JSON.stringify() on a Mongoose document, because JSON.stringify() calls toJSON(). Pass { virtuals: true } to either toObject() or toJSON().
{% endblockquote %}

문서에 따라 모델에 옵션을 추가합니다.

{% codeblock test.js lang:JavaScript %}
    const TestSchema = new Schema({
        str: {
            type: String,
            alias: 'txt'
        },
        num: {
            type: Number,
            alias: 'int'
        }
    }, {
        collection: 'test',
        toJSON: {
            virtuals: true
        }
    });
{% endcodeblock %}

이렇게 하면 GET 요청의 응답이 아래와 같은 형태로 받게 됩니다.

{% codeblock %}
    {
        "_id": "6086d2de1d846b1490f84474",
        "str": "a",
        "num": 1,
        "__v": 0,
        "txt": "a",
        "int": 1,
        "id": "6086d2de1d846b1490f84474"
    }
{% endcodeblock %}

여기서 의문이 생겼었는데 **왜 하필 response 응답을 보낼 때 일까?** 였습니다.

find를 통해 조회한 값을 console.log() 로 출력해보면 alias 값은 포함되어 있지 않습니다.
그런데 그 객체를 res.send() 를 통해 응답을 보내면 위와 같이 alias 값이 포함되어 있습니다.

{% codeblock app.js lang:JavaScript %}
    app.get('/', async (req, res) => {
        try {
            const result = await Test.findOne(
                Test.translateAliases({
                    txt: 'a'
                })
            );
            console.log(result); // { _id: 6086d2de1d846b1490f84474, str: 'a', num: 1, __v: 0 }
            res.send(result);
        } catch (err) {
            console.error(err);
            res.sendStatus(500);
        }
    });
{% endcodeblock %}

이것에 대한 답은 express의 reponse 문서에서 찾을 수 있었습니다.

{% blockquote Express Docs https://expressjs.com/ko/api.html#res.send %}
    When the parameter is an Array or Object, Express responds with the JSON representation
{% endblockquote %}

{% blockquote Express Docs https://expressjs.com/ko/api.html#res.json %}
    Sends a JSON response. This method sends a response (with the correct content-type) that is the parameter converted to a JSON string using JSON.stringify().
{% endblockquote %}

res.send() 를 통해 응답을 보낼 때 보내는 데이터의 타입이 객체(혹은 배열)인 경우 내부적으로 res.json() 을 호출하게 됩니다.
res.json() 에서는 `JSON.stringify()` 를 이용해 응답을 보냅니다. 그렇기 때문에 위의 Mongoose 세팅대로 alias 값이 포함되게 되는 것입니다.

# 5. 원본 값 제거하기

일단 alias 값을 리턴 받는 것까지는 성공했습니다.
그런데 원본 값까지 리턴 받는 것이 아무리 봐도 찝찝합니다.
이제부터 이걸 제거해보도록 하겠습니다.

이 방법이 세련된 방법인지는 모르겠습니다만 어쨌든 제가 찾은 방법은 `transform`을 이용하는 방법입니다.

{% blockquote Mongoose Docs https://mongoosejs.com/docs/api.html#document_Document-toJSON %}
    We may need to perform a transformation of the resulting object based on some criteria, say to remove some sensitive information or return a custom object. In this case we set the optional transform function.

    Transform functions receive three arguments

    `function (doc, ret, options) {}`

    - doc The mongoose document which is being converted
    - ret The plain object representation which has been converted
    - options The options in use (either schema options or the options passed inline)
{% endblockquote %}

위의 내용에 따라 아래와 같이 작성합니다.

{% codeblock test.js lang:JavaScript %}
    const TestSchema = new Schema({
        str: {
            type: String,
            alias: 'txt'
        },
        num: {
            type: Number,
            alias: 'int'
        }
    }, {
        collection: 'test',
        toJSON: {
            virtuals: true,
            transform(doc, ret, options) {
                delete ret.str;
                delete ret.num;
            }
        }
    });
{% endcodeblock %}

이 상태에서 API 요청을 보내면 아래와 같이 원본 이름을 제외한 alias로 결과 값을 받게 됩니다.

{% codeblock lang:JavaScript %}
    {
        "_id": "6086d2de1d846b1490f84474",
        "__v": 0,
        "txt": "a",
        "int": 1,
        "id": "6086d2de1d846b1490f84474"
    }
{% endcodeblock %}

# 6. 한계

지금까지의 방법을 적용하면 alias로 데이터를 조회할수도 있고 API 리턴 값도 받을 수 있습니다.
그런데 한 가지 해결하지 못한 부분이 있습니다.

바로 특정 필드만 조회할 때 alias를 적용하는 것입니다.

{% codeblock app.js lang:JavaScript %}
    const result = await Test.findOne(
        Test.translateAliases({
            txt: 'a'
        }),
        'txt' // alias
    );
{% endcodeblock %}

위와 같이 조회하면 alias로 정상적으로 조회는 됐지만 없는 필드를 읽으려고 하기 때문에 결과 값은 아래와 같이 아무 필드도 받지 못하게 됩니다.

{% codeblock lang:JavaScript %}
    {
        "_id": "6086d2de1d846b1490f84474",
        "id": "6086d2de1d846b1490f84474"
    }
{% endcodeblock %}

따라서 특정 필드만 조회하기 위해선 어쩔 수 없이 원본 이름을 사용해야 합니다.

{% codeblock app.js lang:JavaScript %}
    const result = await Test.findOne(
        Test.translateAliases({
            txt: 'a'
        }),
        'str' // 원본 필드 이름
    );
{% endcodeblock %}

{% codeblock lang:JavaScript %}
    {
        "_id": "6086d2de1d846b1490f84474",
        "txt": "a",
        "id": "6086d2de1d846b1490f84474"
    }
{% endcodeblock %}

계속 찾아봤지만 이 부분까지 alias를 이용하는 방법은 찾지 못했습니다.

해결 방법을 아시는 분은 댓글로 남겨주시면 감사하겠습니다.