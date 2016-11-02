---
layout: post
title:  "代码高亮测试"
date:   2016-11-01 11:15:23
categories: Java
tags: Java
---
{% highlight java linenos %}
/**
 * This is about <code>ClassName</code>.
 * {@link com.yourCompany.aPackage.Interface}
 * @author author
 * @deprecated use <code>OtherClass</code>
 */
public class ClassName<E> extends AnyClass implements InterfaceName<String> {
	enum Color { RED, GREEN, BLUE };
	/* This comment may span multiple lines. */
	static Object staticField;
	// This comment may span only this line
	private E field;
	private AbstractClassName field2;
	// TASK: refactor
	@SuppressWarnings(value="all")
	public int foo(Integer parameter) {
		abstractMethod(inheritedField);
		int local= 42*hashCode();
		staticMethod();
		return bar(local) + parameter;
	}
}
{% endhighlight %}
测试_斜体_测试 Java