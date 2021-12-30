# CSP Webpack Plugin

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![npm version](https://badge.fury.io/js/@melloware%2Fcsp-webpack-plugin.svg)](https://badge.fury.io/js/@melloware%2Fcsp-webpack-plugin)
[![Code Style](https://img.shields.io/badge/code%20style-prettier-brightgreen.svg)](https://github.com/prettier/prettier)
[![Build Status](https://github.com/melloware/csp-webpack-plugin/actions/workflows/build.yml/badge.svg)](https://github.com/melloware/csp-webpack-plugin/actions/workflows/build.yml)

## About

This plugin was forked from the wonderful work done by [Slack](https://github.com/slackhq/csp-html-webpack-plugin) but adds some key features:

- Generate SHA hashes for external and internal JS files
- Configure NONCE for pre-loaded scripts
- Special handling for [PrimeReact](https://www.primefaces.org/primereact/) inline CSS styles
- Typescript definition

## Description

This plugin will generate meta content for your [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
tag and input the correct data into your HTML template, generated by [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin/).

All inline JS and CSS will be hashed and inserted into the policy.

## Installation

Install the plugin with npm:

```bash
npm i --save-dev @melloware/csp-webpack-plugin
```

## Basic Usage

Include the following in your webpack config:

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CspHtmlWebpackPlugin = require('@melloware/csp-webpack-plugin');

module.exports = {
  // rest of webpack config

  plugins: [
    new HtmlWebpackPlugin()
    new CspHtmlWebpackPlugin({
      // config here, see below
    })
  ]
}
```

## Recommended Configuration

By default, the `@melloware/csp-webpack-plugin` has a very lax policy. You should configure it for your needs.

A good starting policy would be the following:

```
new CspHtmlWebpackPlugin({
  'script-src': '',
  'style-src': ''
});
```

Although we're configuring `script-src` and `style-src` to be blank, the CSP plugin will scan your HTML
generated in `html-webpack-plugin` for external/inline script and style tags, and will add the appropriate
hashes and nonces to your CSP policy. This configuration will also add a `base-uri` and `object-src` entry
that exist in the default policy:

```
<meta http-equiv="Content-Security-Policy" content="
  base-uri 'self';
  object-src 'none';
  script-src 'sha256-0Tumwf1AbPDHZO4kdvXUd4c5PiHwt55hre+RDxj9O3Q='
             'nonce-hOlyTAhW5QI5p+rv9VUPZg==';
  style-src 'sha256-zfLUTOi9wwJktpDIoBZQecK4DNIVxW8Tl0cadROvQgo='
">
```

This configuration should work for most use cases, and will provide a strong layer of extra security.

## All Configuration Options

### `CspHtmlWebpackPlugin`

This `CspHtmlWebpackPlugin` accepts 2 params with the following structure:

- `{object}` Policy (optional) - a flat object which defines your CSP policy. Valid keys and values can be found on the [MDN CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) page. Values can either be a string, or an array of strings.
- `{object}` Additional Options (optional) - a flat object with the optional configuration options:
  - `{boolean|Function}` enabled - if false, or the function returns false, the empty CSP tag will be stripped from the html output.
    - The `htmlPluginData` is passed into the function as it's first param.
    - If `enabled` is set the false, it will disable generating a CSP for all instances of `HtmlWebpackPlugin` in your webpack config.
  - `{string}` hashingMethod - accepts 'sha256', 'sha384', 'sha512' - your node version must also accept this hashing method.
  - `{object}` hashEnabled - a `<string, boolean>` entry for which policy rules are allowed to include hashes
  - `{object}` nonceEnabled - a `<string, boolean>` entry for which policy rules are allowed to include nonces
  - `{Function}` processFn - allows the developer to overwrite the default method of what happens to the CSP after it has been created
    - Parameters are:
      - `builtPolicy`: a `string` containing the completed policy;
      - `htmlPluginData`: the `HtmlWebpackPlugin` `object`;
      - `$`: the `cheerio` object of the html file currently being processed
      - `compilation`: Internal webpack object to manipulate the build

### `HtmlWebpackPlugin`

The plugin also adds a new config option onto each `HtmlWebpackPlugin` instance:

- `{object}` cspPlugin - an object containing the following properties:
  - `{boolean}` enabled - if false, the CSP tag will be removed from the HTML which this HtmlWebpackPlugin instance is generating.
  - `{object}` policy - A custom policy which should be applied only to this instance of the HtmlWebpackPlugin
  - `{object}` hashEnabled - a `<string, boolean>` entry for which policy rules are allowed to include hashes
  - `{object}` nonceEnabled - a `<string, boolean>` entry for which policy rules are allowed to include nonces
  - `{Function}` processFn - allows the developer to overwrite the default method of what happens to the CSP after it has been created
    - Parameters are:
      - `builtPolicy`: a `string` containing the completed policy;
      - `htmlPluginData`: the `HtmlWebpackPlugin` `object`;
      - `$`: the `cheerio` object of the html file currently being processed
      - `compilation`: Internal webpack object to manipulate the build

### Order of Precedence:

You don't have to include the same policy / `hashEnabled` / `nonceEnabled` configuration object in both `HtmlWebpackPlugin` and `CspHtmlWebpackPlugin`.

- Config included in `CspHtmlWebpackPlugin` will be applied to all instances of `HtmlWebpackPlugin`.
- Config included in a single `HtmlWebpackPlugin` instantiation will only be applied to that instance.

In the case where a config object is defined in multiple places, it will be merged in the order defined below, with former keys overriding latter. This means entries for a specific rule will not be merged; they will be replaced.

```
> HtmlWebpackPlugin cspPlugin.policy
> CspHtmlWebpackPlugin policy
> CspHtmlWebpackPlugin defaultPolicy
```

## Appendix

#### Default Policy:

```js
{
  'base-uri': "'self'",
  'object-src': "'none'",
  'script-src': ["'unsafe-inline'", "'self'", "'unsafe-eval'"],
  'style-src': ["'unsafe-inline'", "'self'", "'unsafe-eval'"]
};
```

#### Default Additional Options:

```js
{
  enabled: true
  hashingMethod: 'sha256',
  hashEnabled: {
    'script-src': true,
    'style-src': true
  },
  nonceEnabled: {
    'script-src': true,
    'style-src': true
  },
  processFn: defaultProcessFn
}
```

#### Full Default Configuration:

```js
new HtmlWebpackPlugin({
  cspPlugin: {
    enabled: true,
    policy: {
      'base-uri': "'self'",
      'object-src': "'none'",
      'script-src': ["'unsafe-inline'", "'self'", "'unsafe-eval'"],
      'style-src': ["'unsafe-inline'", "'self'", "'unsafe-eval'"]
    },
    hashEnabled: {
      'script-src': true,
      'style-src': true
    },
    nonceEnabled: {
      'script-src': true,
      'style-src': true
    },
    processFn: defaultProcessFn  // defined in the plugin itself
  }
});

new CspHtmlWebpackPlugin({
  'base-uri': "'self'",
  'object-src': "'none'",
  'script-src': ["'unsafe-inline'", "'self'", "'unsafe-eval'"],
  'style-src': ["'unsafe-inline'", "'self'", "'unsafe-eval'"]
}, {
  enabled: true,
  hashingMethod: 'sha256',
  hashEnabled: {
    'script-src': true,
    'style-src': true
  },
  nonceEnabled: {
    'script-src': true,
    'style-src': true
  },
  processFn: defaultProcessFn  // defined in the plugin itself
})
```
## Advanced Usage
### Generating a file containing the CSP directives

Some specific directives require the CSP to be sent to the client via a response header (e.g. `report-uri` and `report-to`)
You can set your own `processFn` callback to make this happen.

#### nginx

In your webpack config:

```js
const RawSource = require('webpack-sources').RawSource;

function generateNginxHeaderFile(
  builtPolicy,
  _htmlPluginData,
  _obj,
  compilation
) {
  const header =
    'add_header Content-Security-Policy "' +
    builtPolicy +
    '; report-uri /csp-report/ ";';
  compilation.emitAsset('nginx-csp-header.conf', new RawSource(header));
}

module.exports = {
  {...},
  plugins: [
    new CspHtmlWebpackPlugin(
      {...}, {
      processFn: generateNginxHeaderFile
    })
  ]
};
```
In your nginx config:
```nginx
location / {
  ...
  include /path/to/webpack/output/nginx-csp-header.conf
}
```
## Contribution

Contributions are most welcome! Please see the included contributing file for more information.

## License

This project is licensed under MIT. Please see the included license file for more information.
