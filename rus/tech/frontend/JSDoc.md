понедельник, 12 октября 2015 г. в 10:13:20

Скопирую [статью](http://www.thinkingmedia.ca/2013/08/valid-jsdoc-annotations-supported-by-phpstorm/) про аннотации для javascript - [JSDoc](http://usejsdoc.org/). Порой бывает очень полезно для автокомплита и прыгания в нужные места в IDE..

## Общие тэги

```
/** 
  * @deprecated Устаревшая функция 
  * @see nameSpace.ClassName#methodName Note, there are no { } brackets. 
  * @see namespace.ClassName Note, there are no { } brackets. 
 **//** This is a comment of how {@link nameSpace.ClassName} is similar to this object. **/
```

### @param

```
/** 
 * @param {paramType} paramName Basic parameter description. 
 * @param {String|Number} paramName Can be one of these types.
 * @param {String} [paramName] Parameter is optional. 
 * @param {String[]} paramName Parameter is an array of strings. 
 * @param {function(string)} paramName Parameter is a function callback. 
 * @param {String} [paramName="mathew"] Parameter has a default value of "mathew" via code. (must be optional). 
 * @param objectName This parameter is an object with values. 
 * @param objectName.property1 This is a property of above object. 
 **/
```

## Inline Documentation

```
/** Define parameter types in function arguments (useful for inline functions) **/
function(/**String**/arg1) { }

/** @type {string} msg **/ Handy for unknown return types.
var msg = foo();
```

### Классы

```
/**
  * @class {namespace.ClassName} Defines a reference to a class. 
  * @augments {namespace.ClassName} Define a derived base class. 
  * @extends alias for @augments 
  * @lends {namespace.ClassName} Declares a block to define part of a class. 
  * @constructor Defines a function as the creator for a class. 
  * @constructs Must be used with @lends to define the initialization function. 
  * @private Highlights invalid references to a property in PHPStorm 
  * @protected Highlights invalid references to a property in PHPStorm 
  * @override Defines a function as overriding a function of the same name in a lower class. 
  * @borrows namespace.ClassName#methodName Tells PHPStorm to use the documentation from another class/method for this method.  
  * @memberOf namespace.ClassName Declares something outside of a class block as part of that class. 
  **/
```

### Функции

```
/**
 * @this {type} Defines what object "this" referrers to. 
 * function(/**String**/arg1)Defines the parameters of an inline function.
 * @return{type} The return type.
 * @returns alias of above.
**/
```

### [Пример использования](http://www.thinkingmedia.ca/2013/08/valid-jsdoc-annotations-supported-by-phpstorm/#some-javascript-examples-of-jsdoc)

Backbone Model and Inheritance A typical example using a Backbone model to demonstrate how to declare inheritance.

```
/**
 * This is a model class what be will inherited. 
 * 
 * @class {gems.BaseModel} 
 * @extends {Backbone.Model} 
 **/ 
gems.BaseModel = Backbone.Model.extend( 
    /** 
     * @lends {gems.BaseModel} 
     */ 
    {
        /**
         * This is a property of the class. 
         *
         * @type {string} 
         **/
        message: 'Hello world',

        /**
         * A constructor for this class.
         *
         * @param {Object} attributes
         * @param {Object} options
         * 
         * @constructs
         */
        initialize: function(attributes, options)
        {
        },

        /**
         * This is a virtual method on the model.
         * 
         * @param {string} arg An argument
         * @public
         * @returns {boolean}
         */
        foo: function(arg)
        {
            return false;
        }
    });
```

Теперь модель унаследовавшая BaseModel

```
/**
 * A document model for Gems.
 *
 * @class {gems.DocumentModel}
 * @extends {gems.BaseModel}
 **/
gems.DocumentModel = gems.BaseModel.extend(
    /**
     * @lends {gems.DocumentModel}
     **/
    {
        /**
         * @param {string} arg An argument
         * @see gems.BaseModel#foo
         * @override
         *
         * @returns {boolean}
         **/
        foo: function(arg)
        {
            if(/**something**/)
            {
                return true;
            }
            return gems.BaseModel.prototype.foo.apply(this,[arg]);
        }
    });
```