---
layout: post
title: "AngularJS file upload directive"
date: 2014-5-8 15:57:40
---
之前需要在页面里面上传文件，却不知道如何将获取到的文件传给 Angular 的 model，想来想去最合适的还是用 Angular 的 directive 来实现，写了之后发现很简单，大概就是下面这样。

```coffeescript
app.directive 'fileUploader', ['$parse', ($parse) ->
    return {
        restrict: 'A',
        link: (scope, element, attrs) ->
            element.bind 'change', ->
                input_file = element[0].files[0]
                $parse(attrs.fileUploader)
                .assign(scope, input_file)
                scope.$apply()
    }
]
```

```html
<input type="file" id="file_uploader" file-input="file" />
```

这里我们定义了一个叫做 fileUploader 的 directive，通过在前端模版中把它和 input 绑定，我们可以获取到 input 的内容，element[0].files 就是上传的文件（files 是一个数组，也就是说我们还可以用它来实现多文件上传）。因为我们我们只上传了一个文件，因此上传的文件为 element[0].files[0]，这里我们用到了 $parse 这个模块，它用来解析 html。我们这里用它是需要获取到赋值给 file-uploader 的那个 model，然后在它所在的 scope 内赋值。这样一来，我们的 controller 中的 $scope.file 就成功获取到了上传的文件的信息。除此之外，我们还可以读取到 file 的很多属性，如 size，type 等等，这为对上传的文件做更多的限制提供了很多的便利。

后记：开始使用 Angular 的时候会总是用以前写 jQuery 的思维来写 Angular 的代码，所以会觉得用起来很痛苦。一旦熟悉了 Angular 自己的一些规则，按着它的规则来就会用起来很爽。directive 就是 Angular 用起来需要适应但是会很爽的东西，它能让我们更好的把 controller 和 views 分离，同时还有很好的重用性。
