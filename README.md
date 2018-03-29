# js-api-generator

[![Build Status][ci-img]][ci-url]
[![Code coverage][cov-img]][cov-url]
[![Code style][lint-img]][lint-url]
[![Dependency Status][dep-img]][dep-url]
[![Dev Dependency Status][dev-dep-img]][dev-dep-url]
[![NPM version][npm-ver-img]][npm-url]
[![NPM downloads][npm-dl-img]][npm-url]
[![NPM license][npm-lc-img]][npm-url]

Generate module for API requesting:

* follow CommonJS specification or ES2015
* from easy config file
* with Promise returned
* use axios to send XMLHttpRequest since 2.0.0
  * Wish use fetch? See [tag 1.10.3](https://github.com/poppinlp/js-api-generator/tree/v1.10.3)
* pack by rollup
* could overwrite options as
* e.t.c

## Getting Started

Install via npm:

```shell
npm i js-api-generator
```

Install via yarn:

```shell
yarn add js-api-generator
```

## API Config File

Config file should be in [YAML](http://www.yaml.org/spec/1.2/spec.html) format and could have `api`, `config` and `build` objects.

### `build` field

Specify some options for building the output module.

#### module {String}

Default: `commonjs`

Specify the module spec of output file. Case insensitivity. Supported list:

* CommonJS
* ES2015
* IIFE

#### moduleName {String}

Default: `''`

Specify the module name.

#### withAxios {Boolean}

Default: `true`

Specify the output module includes axios code or not.

### `config` field

Provide global options which will be overwrote by same option in `api` object.

#### baseURL {String}

Default: `''`

Specify the base url for request which will be prefix to `api.url`.

#### method {String}

Default: `get`

Specify request method. Case insensitivity.

#### headers {Object}

Default: `{}`

Specify custom headers for request such as `X-Requested-With`.

#### timeout {Number}

Default: `5000`

Specify the number of milliseconds before request times out.

#### responseType {String}

Default: `json`

Specify the type of response data. Supported list:

* arraybuffer
* blob
* document
* json
* text
* stream

#### withCredentials {Boolean}

Default: `false`

Indicates whether or not cross-site Access-Control requests should be made using credentials

### api field

Define api list. Every option in `config` field could be overwrote.

#### url {String}

The url for requesting. Support dynamic url which means you could use variable in url link.

If the config url is `/user/:uid` and you call this API by `{uid: 123}`, the request url will be `/user/123`.

#### name {String}

The method name for this api.

#### params {Object}

Default: `{}`

Define the request params. For all key-value pair, the key will be used as property name and the value as variable type(or type list) for type checking. `Any` will be used if can't match any type. Supported type list:

* String
* Number
* Boolean
* Array
* Blob
* Object
* Null
* Any

If a key is ended with `?`, the property name won't include the `?` and this key will be treated as optional.

#### body {Object}

Default: `{}`

Define the request body data. It's just like `params` field.

#### succCond {Object}

Default: `{}`

Define key and value to check the response data. The response will be treated as success when response object has same key-value pair for every key-value pair in this object.

#### succRsp {Object | Null}

Default: `null`

Define the response data and type for success response. If provide an object, it should be in `params` field format. Otherwise provide a `null` means all response data will be returned with no type check.

**NOTE**: This only works when response is a plain object, otherwise you'll get the whole response.

#### failRsp {Object | Null}

Default: `null`

Define the response data and type for fail response. It's just like `succRsp` field.

## API Config File Sample

```yml
build:
  module: es2015

api:
-
  url: /test1
  method: put
  name: foobar
  timeout: 10000
  succRsp:
    data1: String
    data2?: Number
  failRsp:
    data3: Array
    data4?:
      - Number
      - String

config:
  baseURL: http://helloworld.com
  succCond:
    code: 0
  headers:
    Content-Type: application/x-www-form-urlencoded
  failRsp:
    message: String
```

## Generated Module

The generated module includes all APIs in config file.

* Each API accepts an object param which contains `params`, `body` and `config` fields.
  * params: use to make up query string in url.
  * body: use to make up request body.
  * config: use to overwrite the options from config file.
* Each API returns a Promise. The status of promise will determined by http status code and `succCond`.
* Each API won't send request again before response or timeout, but the callback will be queued and will be triggered when get response or timeout.
* The callback function accepts an object param which contains `response` and `xhr` fields.
  * response: the response object determined by http status code, `succRsp` or `failRsp`.
  * xhr: the sent XMLHttpReqeust object.
* The output module is in ES2015 syntax. You may transform it yourself if you want.

### Generated Module Usage Sample

These are samples for using generated module.

#### Node env with CommonJS generated module

```js
var api = require('path/to/generated-api-module');

api.yourApiName({
  // data to send
}).then(res => {
  // res object content is determined by `success` config
  // do some stuff for success
}).catch(err => {
  // err object content is determined by `fail` config
  // do some stuff for fail
});
```

#### Browser env with IIFE generated module

```html
<script src="path/to/generated-api-module.js"></script>
<script>
yourModuleName.yourApiName({
    // data to send
}).then(res => {
    // res object content is determined by `success` config
    // do some stuff for success
}).catch(err => {
    // err object content is determined by `fail` config
    // do some stuff for fail
});
</script>
```

## How to use

You could use this package via node module or CLI.

### Node Module

The export of this package in CommonJS is a function. You could just require this, run that function and pass options in.

### CLI

You could see the bin file in `./node_modules/.bin` and use it by `npx js-api-generator --option=value`.

### Options

The follow options are used to generate module, not for config file.

#### config {String}

Yaml config file path. Should be absolute path or relative to your current working directory.

#### output {String}

The file path for generated module. Should be absolute path or relative to your current working directory.

### Package Usage Example

```js
var api = require('js-api-generator');
var code = api({
  config: 'config yaml path',
  output: 'output js path
});
```

## Browser Compatibility

|                             | Chrome | Firefox | Edge | IE                  | Opera | Safari |
| --------------------------- | ------ | ------- | ---- | ------------------- | ----- | ------ |
| Output file                 | 45.0   | 22.0    | Yes  | Not support         | 32    | 10.0   |
| Compile with babel or buble | Yes    | Yes     | Yes  | Depends on polyfill | Yes   | Yes    |

[ci-img]: https://img.shields.io/travis/poppinlp/js-api-generator.svg?style=flat-square
[ci-url]: https://travis-ci.org/poppinlp/js-api-generator
[cov-img]: https://img.shields.io/coveralls/poppinlp/js-api-generator.svg?style=flat-square
[cov-url]: https://coveralls.io/github/poppinlp/js-api-generator?branch=master
[lint-img]: https://img.shields.io/badge/code%20style-handsome-brightgreen.svg?style=flat-square
[lint-url]: https://github.com/poppinlp/eslint-config-handsome
[dep-img]: https://img.shields.io/david/poppinlp/js-api-generator.svg?style=flat-square
[dep-url]: https://david-dm.org/poppinlp/js-api-generator
[dev-dep-img]: https://img.shields.io/david/dev/poppinlp/js-api-generator.svg?style=flat-square
[dev-dep-url]: https://david-dm.org/poppinlp/js-api-generator#info=devDependencies
[npm-ver-img]: https://img.shields.io/npm/v/js-api-generator.svg?style=flat-square
[npm-dl-img]: https://img.shields.io/npm/dm/js-api-generator.svg?style=flat-square
[npm-lc-img]: https://img.shields.io/npm/l/js-api-generator.svg?style=flat-square
[npm-url]: https://www.npmjs.com/package/js-api-generator
