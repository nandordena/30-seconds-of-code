---
title: Run In Worker
tags: serviceWorker,function,run
---

A function to turn a small "heavy task" into a service worker so as not to interfere with the execution of the page.

The function converts the task into a data url that will execute the task as soon as it receives a data object.
This data will be processed in the onmessage event of the task and at the end it can return a result with postMessage into a task.
the function returns a promise that will resolve with the result data.

```js
const startWorker = (task,data) => {
  return new Promise(function(resolve, reject) {
    if(typeof(Worker) != "function") {
      reject();
    }
    task = 'self.onmessage = data =>{data=data.data;'+task+'}';
    var w = new Worker("data:text/plain,"+encodeURI(task));
    w.postMessage(data);
    w.onmessage = function(event) {
      resolve(event.data);
      w.terminate();
      w = undefined;
    }
  });
}
```

```js
startWorker(
  `
    let result = 0;
    data.forEach(a=>{
      result+=a;
    });
    postMessage(result);
  `
  ,[1,2,3,4,5,6,7,8,9,10]
)
.then(result=>{alert(result)}); // 55
```
