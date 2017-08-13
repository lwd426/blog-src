title: kibana的模块依赖机制
date: 2015-05-23 10:15:49
tags: 
- AngularJS
- kibana
categories: AngularJS

---

### 经验
* 必须先把模块使用require('模块路径');
* 进来后，
才能使用require('module').get('新模块名称',[模块1，模块2...])进行定义模块依赖,
* 才能在新模块中把服务、指令、常量、视图注入到代码里。
* 在模块定义的工程中，也要通过require('辅助模块')；来把所有的依赖模块代码加载进门户文件.

<!-- more -->

### 实际代码
比如：kibana的discover.js文件中

```javascript
define(function (require) {
  var _ = require('lodash');
  var angular = require('angular');
  var moment = require('moment');
  var ConfigTemplate = require('utils/config_template');
  var onlyDisabled = require('components/filter_bar/lib/onlyDisabled');
  var filterManager = require('components/filter_manager/filter_manager');
  var getSort = require('components/doc_table/lib/get_sort');
  var rison = require('utils/rison');
  var datemath = require('utils/datemath');

  require('components/notify/notify');
  require('components/timepicker/timepicker');
  require('directives/fixed_scroll');
  require('directives/validate_json');
  require('components/validate_query/validate_query');
  require('filters/moment');
  require('components/courier/courier');
  require('components/index_patterns/index_patterns');
  require('components/state_management/app_state');
  require('services/timefilter');
  require('components/highlight/highlight_tags');

  var app = require('modules').get('apps/discover', [
    'kibana/notify',//只有上方加载了require('components/notify/notify');模块才能在这里声明依赖，才能在app模块内注入Notifier服务
    'kibana/courier',
    'kibana/index_patterns'
  ]);
  app.controller('discover', function ($scope, config, courier, $route, $window, Notifier,
    AppState, timefilter, Promise, Private, kbnUrl, highlightTags, $location, $anchorScroll, $modal, $modalStack, $log) {
```