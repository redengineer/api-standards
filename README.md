# Xiaohongshu Web API Standards

* [Guidelines](#guidelines)
* [Pragmatic REST](#pragmatic-rest)
* [RESTful URLs](#restful-urls)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [Error handling](#error-handling)
* [Versions](#versions)
* [Record limits](#record-limits)
* [Request & Response Examples](#request--response-examples)
* [Mock Responses](#mock-responses)
* [JSONP](#jsonp)

## 指导原则

本文档提供了小红书API开发过程中的知道原则和示例，鼓励一致性，可维护和借鉴行业内最佳实践，是完全RESTful API定义和开发效率的一个平衡。

## Pragmatic REST

本指导原则目标达到完全Restful, 除了以下几个例外：
* 版本号必须放在URL中, 如下
    * http://www.xiaohongshu.com/api/v1/magazines

## RESTful URLs

### 通用原则
* 每一个URL必须代表一种资源 (resource)
* URL只能有名词，不能含有动词
* 为了一致，名词应该使用复数
* 对于资源的具体操作，使用HTTP动词 (GET, POST, PUT, DELETE)
* URL不应该比resource/identifier/resource更深
* 将版本号放在URL路径的第一层, 如http://www.xiaohongshu.com/v1/path/to/resource.
* 指定返回字段的话用逗号隔开

### 协议
API与用户的通信协议，总是使用HTTPS. (TBD)

### 格式(Formating)
服务器返回的数据格式使用JSON.

### 好的例子
* List of magazines:
    * GET http://www.xiaohongshu.com/api/v1/magazines
* Filtering is a query:
    * GET http://www.xiaohongshu.com/api/v1/magazines?year=2011&sort=desc
    * GET http://www.xiaohongshu.com/api/v1/magazines?topic=economy&year=2011
* A single magazine in JSON format:
    * GET http://www.xiaohongshu.com/api/v1/magazines/1234
* All articles in (or belonging to) this magazine:
    * GET http://www.xiaohongshu.com/api/v1/magazines/1234/articles
* Specify optional fields in a comma separated list:
    * GET http://www.xiaohongshu.com/api/v1/magazines/1234?fields=title,subtitle,date
* Add a new article to a particular magazine:
    * POST http://www.xiaohongshu.com/api/v1/magazines/1234/articles

### 坏的例子
* 非复数:
    * http://www.xiaohongshu.com/magazine
    * http://www.xiaohongshu.com/magazine/1234
    * http://www.xiaohongshu.com/publisher/magazine/1234
* URL中含动词:
    * http://www.xiaohongshu.com/magazine/1234/create
* 非query中的过滤
    * http://www.xiaohongshu.com/magazines/2011/desc

## HTTP动词

HTTP verbs, or methods, should be used in compliance with their definitions under the [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) standard.
The action taken on the representation will be contextual to the media type being worked on and its current state. Here's an example of how HTTP verbs map to create, read, update, delete operations in a particular context:

| HTTP METHOD | POST            | GET       | PUT         | DELETE |
| ----------- | --------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE          | READ      | UPDATE      | DELETE |
| /dogs       | Create new dogs | List dogs | Bulk update | Delete all dogs |
| /dogs/1234  | Error           | Show Bo   | If exists, update Bo; If not, error | Delete Bo |

(Example from Web API Design, by Brian Mulloy, Apigee.)

### 过滤信息 (Filtering)
如果记录数量很多，服务器不可能都将他们返回给用户。API应该提供参数，过滤返回结果。

下面是一些常见 的参数：
* ?limit=10: 指定返回记录的数量
* ?offset=10: 指定返回记录的开始位置
* ?page=2&per_page=100: 指定第几页，以及每页的记录数
* ?sortby=name&order=asc: 指定返回结果按照哪个属性排序，以及排序顺序
* ?animal_type_id=1: 指定筛选条件

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals与/animals?zoo_id=ID的含义是相同的。

## 返回结果

* No values in keys
* No internal-specific names (e.g. "node" and "taxonomy term")
* Metadata should only contain direct properties of the response set, not properties of the members of the response set

### Good examples

No values in keys:

    "tags": [
      {"id": "125", "name": "Environment"},
      {"id": "834", "name": "Water Quality"}
    ],


### Bad examples

Values in keys:

    "tags": [
      {"125": "Environment"},
      {"834": "Water Quality"}
    ],


## 错误处理 (Error handling)

Error responses should include a common HTTP status code, message for the developer, message for the end-user (when appropriate), internal error code (corresponding to some specific internally determined ID), links where developers can find more info. For example:

    {
      "status" : 400,
      "developerMessage" : "Verbose, plain language description of the problem. Provide developers
       suggestions about how to solve their problems here",
      "userMessage" : "This is a message that can be passed along to end-users, if needed.",
      "errorCode" : "444444",
      "moreInfo" : "http://www.xiaohongshu.com/developer/path/to/help/for/444444,
       http://drupal.org/node/444444",
    }

Use three simple, common response codes indicating (1) success, (2) failure due to client-side problem, (3) failure due to server-side problem:
* 200 - OK
* 400 - Bad Request
* 500 - Internal Server Error


## 版本 (Versioning)

* 发布API必须含有版本号
* 版本号应该是整数，不能含有小数，以"v"开头，如：
    * 好的: v1, v2, v3
    * 不好的: v-1.1, v1.2, 1.3

## 返回限制(Record limits)

* If no limit is specified, return results with a default limit.
* To get records 51 through 75 do this:
    * http://www.xiaohongshu.com/magazines?limit=25&offset=50
    * offset=50 means, ‘skip the first 50 records’
    * limit=25 means, ‘return a maximum of 25 records’

Information about record limits and total available count should also be included in the response. Example:

    {
        "metadata": {
            "resultset": {
                "count": 227,
                "offset": 25,
                "limit": 25
            }
        },
        "results": []
    }

## Request & Response Examples

### API Resources

  - [GET /magazines](#get-magazines)
  - [GET /magazines/[id]](#get-magazinesid)
  - [POST /magazines/[id]/articles](#post-magazinesidarticles)

### GET /magazines

Example: http://www.xiaohongshu.com/api/v1/magazines

Response body:

    {
        "metadata": {
            "resultset": {
                "count": 123,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "id": "1234",
                "type": "magazine",
                "title": "Public Water Systems",
                "tags": [
                    {"id": "125", "name": "Environment"},
                    {"id": "834", "name": "Water Quality"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Elementary"},
                    {"id": "834", "name": "Charter Schools"}
                ],
                "created": "126251302"
            }
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Pre-school"},
                ],
                "created": "126251302"
            }
        ]
    }

### GET /magazines/[id]

Example: http://www.xiaohongshu.com/api/v1/magazines/[id]

Response body:

    {
        "id": "1234",
        "type": "magazine",
        "title": "Public Water Systems",
        "tags": [
            {"id": "125", "name": "Environment"},
            {"id": "834", "name": "Water Quality"}
        ],
        "created": "1231621302"
    }



### POST /magazines/[id]/articles

Example: Create – POST  http://www.xiaohongshu.com/api/v1/magazines/[id]/articles

Request body:

    [
        {
            "title": "Raising Revenue",
            "author_first_name": "Jane",
            "author_last_name": "Smith",
            "author_email": "jane.smith@example.gov",
            "year": "2012",
            "month": "August",
            "day": "18",
            "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam eget ante ut augue scelerisque ornare. Aliquam tempus rhoncus quam vel luctus. Sed scelerisque fermentum fringilla. Suspendisse tincidunt nisl a metus feugiat vitae vestibulum enim vulputate. Quisque vehicula dictum elit, vitae cursus libero auctor sed. Vestibulum fermentum elementum nunc. Proin aliquam erat in turpis vehicula sit amet tristique lorem blandit. Nam augue est, bibendum et ultrices non, interdum in est. Quisque gravida orci lobortis... "
        }
    ]


## Mock Responses
It is suggested that each resource accept a 'mock' parameter on the testing server. Passing this parameter should return a mock data response (bypassing the backend).

Implementing this feature early in development ensures that the API will exhibit consistent behavior, supporting a test driven development methodology.

Note: If the mock parameter is included in a request to the production environment, an error should be raised.


## JSONP

JSONP is easiest explained with an example. Here's one from [StackOverflow](http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top):

> Say you're on domain abc.com, and you want to make a request to domain xyz.com. To do so, you need to cross domain boundaries, a no-no in most of browserland.

> The one item that bypasses this limitation is `<script>` tags. When you use a script tag, the domain limitation is ignored, but under normal circumstances, you can't really DO anything with the results, the script just gets evaluated.

> Enter JSONP. When you make your request to a server that is JSONP enabled, you pass a special parameter that tells the server a little bit about your page. That way, the server is able to nicely wrap up its response in a way that your page can handle.

> For example, say the server expects a parameter called "callback" to enable its JSONP capabilities. Then your request would look like:

>         http://www.xyz.com/sample.aspx?callback=mycallback

> Without JSONP, this might return some basic javascript object, like so:

>         { foo: 'bar' }

> However, with JSONP, when the server receives the "callback" parameter, it wraps up the result a little differently, returning something like this:

>         mycallback({ foo: 'bar' });

> As you can see, it will now invoke the method you specified. So, in your page, you define the callback function:

>         mycallback = function(data){
>             alert(data.foo);
>         };

http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top

### 参考文档
* [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
* [White House API Standards](https://github.com/WhiteHouse/api-standards)
* [Designing HTTP Interfaces and RESTful Web Services](https://www.youtube.com/watch?v=zEyg0TnieLg)
* [API Facade Pattern](http://apigee.com/about/resources/ebooks/api-fa%C3%A7ade-pattern), by Brian Mulloy, Apigee
* [Web API Design](http://pages.apigee.com/web-api-design-ebook.html), by Brian Mulloy, Apigee
* [Fielding's Dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
