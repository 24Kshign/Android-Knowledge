### xutils3源码阅读

```
//handleType-->class  获取setContentView方法
Method setContentViewMethod = handlerType.getMethod("setContentView", int.class);
//反射执行该方法
setContentViewMethod.invoke(activity, viewId);
```

```
View view = finder.findViewById(viewInject.value(), viewInject.parentId());
if (view != null) {
	//可以操作所有修饰
	field.setAccessible(true);
	//反射注入属性
	field.set(handler, view);
}
```
**属性注入流程：**

利用反射去获取Annotation--->value--->findViewById--->反射注入属性

**事件注入流程：**

利用反射去获取Annotation--->value--->setOnClickListener--->动态代理反射执行方法