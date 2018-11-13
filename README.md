- [Inheritance](#proto)
- [Proxy](#proxy)
---

# 3 Kinds of Prototypal Inheritance <a name="proto"></a>
## Functional Inheritance for private data

```js
const fakeClassObject = () =>{
  const _Data ={
    private:{//private key protected
      id: Math.round(Math.random() * 10000),
    },
    public:{}
  }
  return Object.assign({},this, {
    //public
    set(name,value) {
      _Data.public[name] = value
    },
    get(name) {
      return _Data.public[name]
    },
    //private
    getInfo() {
      return `
      Name: ${_Data.public['name']} 
      ID: ${_Data.private['id']}`
    }
  })
}
const fakeClass = () => fakeClassObject.call()

const mark = fakeClass()
const tony = fakeClass()
mark.set('name','Mark')
tony.set('name','Tony')
//or just...
const bill = fakeClassObject.call()
bill.set('name', 'Bill')
console.log(tony.getInfo())
console.log(bill.getInfo())
console.log(mark.getInfo())
/**
 * 
      Name: Tony 
      ID: 3757

      Name: Bill 
      ID: 7034
      
      Name: Mark 
      ID: 6160

 */


```

## Delegation / Differential Inheritance

```js
const obj1 ={
  name: 'obj1',
  unique: 'still 1'
  
}
const obj2 ={
  name: 'obj2',
  
}
Object.assign(obj1,obj2)//2 overwites name in obj1
console.log(obj1)
/**
 Object {name: "obj2", unique: "still 1"}
*/
```

## Concatenative Inheritance / Cloning / Mixins
```js

const obj1 ={
  name: 'obj1',
  unique: 'still 1'
  
}
const obj2 ={
  name: 'obj2',
}
const newObj = {}
Object.assign(newObj, obj1, obj2)
console.log(obj1)//remains unchanged
console.log(newObj)// new mix
/**
 Object {name: "obj1", unique: "still 1"}
 Object {name: "obj2", unique: "still 1"}

*/

```

# Proxy <a name="proxy"></a>

```js

//Private data using Proxy

const proxied =  () => {
  var target = {
    _prop: {id: Math.round(Math.random() * 1000)}
  }
  var handler = {
    get (target, key) {
      invariant(key, 'get')
      return key.search(/id/i) === 0 ? target._prop[key] : target[key]
    },
    set (target, key, value) {
      invariant(key, 'set')
      target[key] = value
      return true
    },
    has(target, key){
      if (key[0] === '_') {
        return false
      }
      return key in target
    },
    deleteProperty (target, key) {
      invariant(key, 'delete')
      return true
    },
    defineProperty (target, key, descriptor) {
      invariant(key, 'define')
      return true
    }
  }
  return new Proxy(target, handler)
}
const invariant = (key, action) =>{
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`)
  }
}

let test = proxied()
try{
  test.a = 'b'
  console.log(test.a)
  console.log(`is _prop in test? ${'_prop' in test}`)//<< cannot even see it
  console.log(`ID: ${test.id}`) //private id accessible via get handler 
  test.id = 'changed id'//allowed..
  console.log(`ID: ${test.id} still`)//but won't be accessible, superceded by private
  for (let key in test) {// cannot hide from loops though - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/enumerate
    console.log(`test[${key}]`)
  }
  test._prop = 'test' //Throw - cannot set it
  //test._prop //Throw -  cannot access it
  //delete test._prop //Throw - cannot delete it
}
catch(e){
  console.log(e)
}
/**

b
is _prop in test? false
ID: 590
ID: 590 still
test[_prop]
test[a]
test[id]
Error: Invalid attempt to set private "_prop" property

*/

```
