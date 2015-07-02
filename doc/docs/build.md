## 工作原理

FIS3 是基于文件对象进行构建的，每个进入 FIS3 的文件都会实例化成一个 File 对象，整个构建过程都对这个对象进行操作完成构建任务。以下通过伪码来阐述 FIS3 的构建流程。

### 构建流程

```js
fis.release = function () {
  var src = fis.util.find(fis.project.root);
  var files = {};
  src.forEach(function (f) {
    var file = new File(f);
    files[file.subpath] = fis.compile(file);
  });
  pipe('prepackager', files);
  pipe('packager', files);
  pipe('spriter', files);
  pipe('postpackager', files);
}
```

### 单文件编译流程

```js
fis.compile = function (file) {
  if (file.isFile()) {
    if (!file.hasCache) {
      process(file);
    } else {
      file.revertCache();
    }
  } else {
    process(file);
  }
};

function process(file) {
  if (file.lint) {
    pipe('lint', file);
  }
  if (file.parser) {
    pipe('parser', file);
  }
  if (file.preprocessor) {
    pipe('preprocessor', file);
  }
  if (file.useStandard) {
    standard(file); // 标准化处理
  }
  if (file.postprocessor) {
    pipe('postprocessor', file);
  }
  if (file.optimizer) {
    pipe('optimizer', file);
  }
}
```

### File 对象

```js
function File(filepath) {
  var props = path.info(filepath);
  merge(props, fis.matchRules(filepath)); // merge 分配到的属性
  assign(this, props); // merge 属性到对象
}
```