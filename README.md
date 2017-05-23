# mongoose toJSON project

`toJSON` projection plugin for mongoose.

Adds advanced field selection capabilities to `toJSON` with optional predefined levels and level selector capabilities.

## Installation

```bash
$ npm install mongoose-to-json-project
```

## Basic Usage
Register this plugin last as it will bootstrap any previous transform function option.

```javascript
schema.plugin(require('mongoose-to-json-project'), {
  // your predefined levels:
  levels: {
    // public level
    public: 'some.deep.field.to.include -some.deep.field.to.exclude',
    // private level
    private: '-password',
    // system level
    system: ''
  },
  // (optional) default level as a String
  level: 'public',
  //  or a synchronous level selector function (preferred method).
  // All transform options are passed on to level selector functions,
  // e.g. we can get the user that requested the transform from `options.user`
  // if user is passed into `toJSON` options, like `toJSON({ user: req.user })`.
  level: (doc, ret, options) => doc._id.equals(options.user._id) ? 'private' : 'public'
});
```

### Schema Tree Level Definitions
It is possible to define level inclusions and exclusions in the schema declaration.<br>**Mixing inclusions and exclusions are prohibited and throws an error.**<br>**Inclusion levels must be predefined in plugin level options.**

```javascript
const schema = new Schema({
    myInvisibleField: {
      type: String,
      level: '-public -private' // visible on levels other than 'public' and 'private'
    },
    myInternalField: {
      type: String,
      level: 'internal' // only visible on 'internal' level, 'internal' must be predefined.
    }
});

schema.plugin(require('mongoose-to-json-project'), {
  levels: {
      'internal': '' // predefintion
  }
});
```

## Plugin Options

```javascript
schema.plugin(require('mongoose-to-json-project'), options);
```

### `levels: Object`
Predefined levels to use with a level selector. Key is level name. Value is a Mongoose style dot-notation, space delimited projection. Mixing inclusions and exclusions is possible. Defining at least one inclusion causes all other fields to be excluded automatically like MongoDB. Child inclusions precedes Parent exclusions, e.g. '-parent parent.child1' results in all properties but parent.child1 to be excluded.

```javascript
levels: {
    public: 'username rank status',
    private: '-password -secretField -secret.deep.field'
}
```

### `level: String`
Basic static level selector.

```javascript
level: 'public'
```

### `level: Function(doc, ret, options)`
Synchronous function that must return level name to use as a string. **Preferred level selector method!**
- `doc` - Mongoose Document
- `ret` - Document as plain Object
- `options` - Transform options (Same as `obj` - if specified)

```javascript
level: (doc, ret, options) => doc._id.equals(options.user._id) ? 'private' : 'public'
```

# Mongoose API Extensions and Modifications

## Document#toJSON(`obj`)<a name="toJSON"></a>
Added options:

`obj.projection: String` - Mongoose style dot-notation, space delimited projection argument. Both inclusions and exclusions are possible but inclusions takes precedence thus excluding all other fields.

`obj.level: String` - Set level to use. (!)

`obj.level: Function(doc, ret, options)` - Synchronous function that must return level name to use as a string. (!)
- `doc` - Mongoose Document
- `ret` - Document as plain Object
- `options` - Transform options (Same as `obj` - if specified)

(!) **CAUTION:**  
The level option is passed to the toJSON method call of populated subdocuments and it may not be compatible. Therefore use with caution and if possible, avoid level option completely and depend on schema defaults instead. A Function is the preferred level selector method in plugin options.

## Document#set(`path`, `val`, `[type]`, `[options]`)
Added options:  same as [`toJSON`](#toJSON)  
See: [mongoose docs Document#set](http://mongoosejs.com/docs/api.html#document_Document-set)

## Model#toJSONOptionsExtend(`obj`)
This method extends `obj` with the default schema toJSON options (Adds defaults to prototype chain).<br>In mongoose the options passed to `toJSON` do not inherit the defaults, this method solves this.

Example:

```javascript
let options = Model.toJSONOptionsExtend({
    user: req.user,
    projection: '-some.field.to.exclude some.field.to.include'
});
let plainObject = document.toJSON(options);
```

## Model#getLevelSchemaTree(`levelName`)
Returns a clone of the schema tree (plain object) with level projection applied.

# Compatibility
Only intended for Mongoose v4 with NodeJS later than v4. Tested with Mongoose 4.2.8.

# License
[LGPL-3](LICENSE)

Forked from [mongoose-toObject-project](https://github.com/Faleij/mongoose-toObject-project) by Faleij [faleij@gmail.com](mailto:faleij@gmail.com)
