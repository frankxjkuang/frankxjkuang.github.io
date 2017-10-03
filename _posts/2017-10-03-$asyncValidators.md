---
layout: post
title:  angular表单的异步验证
date:   2017-10-03 12:54:00 +0800
categories: document
tag: course
---

* content
{:toc}


在表单验证的时候前端通常只需要做一些同步的数据格式验证，但是异步验也是不可避免的，特别是用户注册的时候。angular的同步验证和异步验证几乎一模一样，只是异步验证的时候需要发送ajax请求，仅此而已，所以这里只记录一下如何使用$asyncValidators实现表单数据的异步验证。

html 部分
====================================

```html
<input type="text"
       name="userName"
       ng-model="userName"
       user-id="user.id"
       user-type="vip"
       user-name-validator>

<div>
    <span class="text-error" ng-show="actionForm.userName.$pending.userNameValidator">用户名验证中...</span>
    <span class="text-error" ng-show="actionForm.userName.$dirty && actionForm.userName.$error.userNameValidator">用户名不可用</span>
</div>
```

js 部分
====================================

```javascript
import App from '../js/app.module';

App.directive('userNameValidator', ['$http', '$q', ($http, $q) => ({
  require: 'ngModel',
  scope: {
    userId: '=',   // 通过 = 的方式获取变量的值
    userType: '@', // 通过 @ 的方式获取字符串的值
    // 此外还可以通过 & 的方式获取一个表达式
  },
  link: (scope, elm, attrs, ctrl) => {
    ngModel.$asyncValidators.userNameValidator = (modelValue, viewValue) => {
      const value = modelValue || viewValue;

      return $http({method: 'GET', url:`${config.DomainPrefix}?usertype=${scope.userType},name=${value}`}).then((response) => {
        if (response.code === 200) {
          if (response.data.functionType === $scope.validatorProjectType) {
            return $q.resolve();
          }
          return $q.reject();
        } else if (response.code === 9999) {
          alert(response.message);
          return $q.reject();
        }

        return $q.reject();
      });
    };
  },
})]);
```

> **PS:**表单元素包含同步验证和异步验证的时候，如果未通过同步验证将不会进行异步验证，而且数据未通过同步验证的情况是不会进行双向数据绑定的，这是一个小的知识点，但是不可忽略。

