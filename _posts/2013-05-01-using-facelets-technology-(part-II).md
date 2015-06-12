---
layout: post
title:  "Using Facelets technology to create custom composite components (part II)"
date:   2013-05-01 17:36:10
categories: [facelets, jsf]
---

The [first part]({% post_url 2013-04-30-using-facelets-technology-(part-I) %}) of this article presented an approach to 
create a custom Facelets component that behaves like a tabbed panel.

This part discusses the creation of a custom table that supports adding any kind of composite columns.

![picture](/images/facelets_pic_1.png)

As you see in the picture, the particularities of this table are the first column, which is made of checkboxes, and the 
last one, made of drop-downs. So, can we build a custom facelet component that can allow something like this?

For this demonstration, the base component is going to be an icefaces component, an ice:dataTable (similar to 
h:dataTable, but AJAX-enabled).

Let’s see how the client code could look like:

{% highlight html %}

<custom:customTable
    id="my-dynamic-table"
    pojoList="#{myBean.somePojoList}"  
    columnsToIncludeBeforeBodyAndProperties="/includes/customColumn1.xhtml:property1,
                                             /includes/customColumn2.xhtml:property2"
    pojoProperties="property3,property4,property5,property6"
</custom:customTable>

{% endhighlight %}

The table can be thought of as having a main ‘body’, comprised of ‘regular’ columns, and a part built out of ‘custom’ 
columns.

The body of the table represents the columns containing cells that render as text properties from the underlying data 
model, the data model being a list of POJOs. Here’s how the ‘body’ is built: iteratively split “pojoProperties” 
parameter by “,”, obtaining a property, render that property value as text inside the column and attach each column to 
the table:

{% highlight html %}

<ice:dataTable
    id="#{id}"
    value="#{pojoList}"
    var="_pojo"
    varStatus="indexer"
 
    <c:forEach
        items="#{fn:split(pojoProperties, ',')}"
        var="_property"
        varStatus="indexer">
 
        <ice:column>
            <ice:outputText value="#{_pojo[_property]}"/>
            <ice:outputLabel>
                this is row number #{indexer.index}
            </ice:outputLabel>
        </ice:column>
 
    </c:forEach>
 
</ice:dataTable>

{% endhighlight %}

As for the part built out of ‘custom’ columns, the innovation would be the fact that you can add any kind and number of 
columns by specifying the ‘address’ (the location of the file) and an additional parameter, the name of a property from 
the bean.

The component will separate the column addresses, splitting them by comma (','). Then, using the ui:include tag, each of 
them will be added before the ‘body’ of the table (hence the name, columnsToInculdeBeforeBodyAndProperties). In the code 
below, the entity instance, the name of the property and the current iteration index (remember that ice:dataTable is an 
iteration component) will be sent to each column specified at the location. Those parameter values can be used inside 
the column file for data processing.

{% highlight html %}

<!-- the ice:dataTable tag declaration is duplicated here, just for clarity; 
the section from the previous code sample defined by the c:forEach tag can  
coexist under this ice:dataTable tag -->
<ice:dataTable
    id="#{id}"
    value="#{pojoList}"
    var="_pojo"
    varStatus="indexer"
 
    <c:forEach
            items="#{fn:split(columnsToIncludeBeforeBodyAndProperties, ',')}"
            var="_columnNameAndProperty">
            <ui:param
                name="columnName"
                value="#{fn:substringBefore(_columnNameAndProperty, ':')}" />
            <ui:param
                name="property"
                value="#{fn:substringAfter(_columnNameAndProperty, ':')}" />
                <c:if test="#{not empty columnName}">
                    <ui:include src="#{columnName}">
                        <ui:param
                            name="pojo"
                            value="#{_pojo}" />
                        <ui:param
                            name="property"
                            value="#{property}" />
                        <ui:param
                            name="index"
                            value="#{indexer.index}" />
                    </ui:include>
                </c:if>
    </c:forEach>
 
<!-- rest of the code omitted -->
 
</ice:dataTable>

{% endhighlight %}

This is how a custom column could look like:

{% highlight html %}

<!--  this content goes into a file named customColumn1.xhtml,  
ui:composition wrapping tag omitted  -->
 
<ice:column>
 
    <ice:outputLabel>
       current pojo has this property value: #{pojo.property}
    </ice:outputLabel>
    <ice:outputLabel>
        this is row number #{index}
    </ice:outputLabel>
 
</ice:column>

{% endhighlight %}

Again, note the flexibility that this approach gives you.