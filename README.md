- [Inheritance](#proto)
- [Proxy](#proxy)
- [ProxyEvent](#proxye)
- [Base64 encode/decode](#encodedecode)
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
  /**
  or using the spread...
  return (...this,
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
    })
  */
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
      return key.search(/id/i) === 0 ? target._prop['id'] : target[key]
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

# ProxyEvent <a name="proxye"></a>
```js
//https://javascript.info/mixins
let eventMixin = {
  /**
   * Subscribe to event, usage:
   *  menu.on('select', function(item) { ... }
  */
  on(eventName, handler) {
    if (!this._eventHandlers) this._eventHandlers = {};
    if (!this._eventHandlers[eventName]) {
      this._eventHandlers[eventName] = [];
    }
    this._eventHandlers[eventName].push(handler);
  },
  /**
   * Cancel the subscription, usage:
   *  menu.off('select', handler)
   */
  off(eventName, handler) {
    let handlers = this._eventHandlers && this._eventHandlers[eventName];
    if (!handlers) return;
    for (let i = 0; i < handlers.length; i++) {
      if (handlers[i] === handler) {
        handlers.splice(i--, 1);
      }
    }
  },
  /**
   * Generate the event and attach the data to it
   *  this.trigger('select', data1, data2);
   */
  trigger(eventName, ...args) {
    if (!this._eventHandlers || !this._eventHandlers[eventName]) {
      return; // no handlers for that event name
    }
    // call the handlers
    this._eventHandlers[eventName].forEach(handler => handler.apply(this, args));
  }
}

const proxyEvent = (args) =>{
  let target = Object.assign({}, args)
  let handler = Object.assign({},  {
    accum : 0,
    setCount : 0,
    getCount : 0,
    get (target, key) {
      this.getCount ++
      return target[key]
    },
    set (target, key, value) {
      if (key === 'value'){
        if (value === -999.25){
          throw new Error(`Illegal value [ ${value} ]`)
        }
        this.setCount ++
        this.accum += value
        console.log(`accumulated value: ${this.accum} from ${this.setCount} sets`)
      }
      target[key] = value
      return true
    },
    has(target, key){
      return key in target
    },
    deleteProperty (target, key) {
      return true
    },
    defineProperty (target, key, descriptor) {
      return true
    }
  })
  return new Proxy(target, handler)
}

//create an object that has some action you wish to emit an event
let myObj = Object.assign({}, eventMixin, {
  value: 0,
  select(value){
    this.value = value
    this.trigger('select', value)//trigger the event
  }
})

const myProxyEvent = proxyEvent(myObj)
myProxyEvent.on('select', value => console.log(`value is ${value}`))
try{
  let count = 5
  while(count--){
    myProxyEvent.select(Math.random() * 10000)
  }
  myProxyEvent.select(-999.25)
}
catch(e){
  console.log(e)
}

/**
accumulated value: 4735.3172779006145 from 1 sets
value is 4735.3172779006145
accumulated value: 8290.485097516677 from 2 sets
value is 3555.167819616063
accumulated value: 10934.188161566253 from 3 sets
value is 2643.7030640495764
accumulated value: 12905.823055446239 from 4 sets
value is 1971.6348938799854
accumulated value: 20261.303036288773 from 5 sets
value is 7355.479980842532
Error: Illegal value [ -999.25 ]
*/
```

# Base64 encode/decode <a name="encodedecode"></a>
```js
/**
 * The "Unicode Problem"
Since DOMStrings are 16-bit-encoded strings, in most browsers calling window.btoa on a
Unicode string will cause a Character Out Of Range exception if a character exceeds the range of a 8-bit byte (0x00~0xFF).

JavaScript's UTF-16 => base64
-----------------------------
A very fast and widely useable way to solve the unicode problem is by encoding JavaScript native UTF-16 strings directly into base64.
This method is particularly efficient because it does not require any type of conversion, except mapping a string into an array. 

The following code is also useful to get an ArrayBuffer from a Base64 string and/or viceversa
 */

"use strict";

class EncodeDecode {
  constructor(encodeThis) {
    this.stringToEncode = encodeThis;
  }
  /**
   * Getters
   */
  get theString() {
    return this.stringToEncode;
  }
  get UTF16CodeUnits() {
    return this.aUTF16CodeUnits;
  }
  get theEncodedString() {
    return this.encodedString;
  }
  get theDecodedString() {
    return this.decodedString;
  }
  /** 
   *  Base64 string to array encoding
   */

  /**
   * Base64 string to array encoding
   * @param {UINT} nUint6 
   */
  uint6ToB64(nUint6) {
    return nUint6 < 26 ?
      nUint6 + 65
      : nUint6 < 52 ?
        nUint6 + 71
        : nUint6 < 62 ?
          nUint6 - 4
          : nUint6 === 62 ?
            43
            : nUint6 === 63 ?
              47
              :
              65;
  }
  /**
   * Array of bytes to base64 string decoding
   * @param {number} nChr character code
   */
  b64ToUint6(nChr) {
    return nChr > 64 && nChr < 91 ?
      nChr - 65
      : nChr > 96 && nChr < 123 ?
        nChr - 71
        : nChr > 47 && nChr < 58 ?
          nChr + 4
          : nChr === 43 ?
            62
            : nChr === 47 ?
              63
              :
              0;
  }

   /**
    * Encode an Array
    * @param {array} aBytes Uint8Array buffer
    */
  base64EncArr(aBytes) {
    let eqLen = (3 - (aBytes.length % 3)) % 3, sB64Enc = "";
    for (let nMod3, nLen = aBytes.length, nUint24 = 0, nIdx = 0; nIdx < nLen; nIdx++) {
      nMod3 = nIdx % 3;
      const uint6ToB64 = this.uint6ToB64;
      /* Uncomment the following line in order to split the output in lines 76-character long: */
      /*
      if (nIdx > 0 && (nIdx * 4 / 3) % 76 === 0) { sB64Enc += "\r\n"; }
      */
      nUint24 |= aBytes[nIdx] << (16 >>> nMod3 & 24);
      if (nMod3 === 2 || aBytes.length - nIdx === 1) {
        sB64Enc += String.fromCharCode(uint6ToB64(nUint24 >>> 18 & 63), uint6ToB64(nUint24 >>> 12 & 63), uint6ToB64(nUint24 >>> 6 & 63), uint6ToB64(nUint24 & 63));
        nUint24 = 0;
      }
      this.encodedString = eqLen === 0 ?
        sB64Enc
        :
        sB64Enc.substring(0, sB64Enc.length - eqLen) + (eqLen === 1 ? "=" : "==");
    }

  }
  /**
   *  Encode to base64 using native UTF-16
   */
  toBase64() {
    let aUTF16CodeUnits = new Uint16Array(this.stringToEncode.length);
    const self = this;
    Array.prototype.forEach.call(aUTF16CodeUnits, function (el, idx, arr) { arr[idx] = self.stringToEncode.charCodeAt(idx); });
    this.aUTF16CodeUnits = aUTF16CodeUnits;
    this.base64EncArr(new Uint8Array(aUTF16CodeUnits.buffer));
  }

  /**
   * Decode the encodedString
   * 
   * @param {number} nBlockSize 
   */
  base64DecToArr(nBlockSize) {
    let
      sB64Enc = this.encodedString.replace(/[^A-Za-z0-9\+\/]/g, ""), nInLen = sB64Enc.length,
      nOutLen = nBlockSize ? Math.ceil((nInLen * 3 + 1 >>> 2) / nBlockSize) * nBlockSize : nInLen * 3 + 1 >>> 2, aBytes = new Uint8Array(nOutLen);

    for (let nMod3, nMod4, nUint24 = 0, nOutIdx = 0, nInIdx = 0; nInIdx < nInLen; nInIdx++) {
      nMod4 = nInIdx & 3;
      nUint24 |= this.b64ToUint6(sB64Enc.charCodeAt(nInIdx)) << 18 - 6 * nMod4;
      if (nMod4 === 3 || nInLen - nInIdx === 1) {
        for (nMod3 = 0; nMod3 < 3 && nOutIdx < nOutLen; nMod3++ , nOutIdx++) {
          aBytes[nOutIdx] = nUint24 >>> (16 >>> nMod3 & 24) & 255;
        }
        nUint24 = 0;
      }
    }
    return aBytes.buffer;
  }

  /**
   * Encode a string as Base64
   */
  encodeString() {
    this.toBase64();
  }

  /**
   * Decode the Base64 encoded string
   */
  decodeString() {
    const nArray = new Uint16Array(this.base64DecToArr(2));
    this.decodedString = String.fromCharCode.apply(null, new Uint16Array(this.base64DecToArr(2)));
  }

}

/* Testing */
// construct with string to encode
const myEncodeDecoder = new EncodeDecode("☸☹☺☻☼☾☿");

myEncodeDecoder.encodeString();
console.log(`Encoded String: ${myEncodeDecoder.theEncodedString}`);
myEncodeDecoder.decodeString();
console.log(`Decoded String: ${myEncodeDecoder.theDecodedString}`);
console.log(` equal? : ${myEncodeDecoder.theString === myEncodeDecoder.theDecodedString}`)
```
