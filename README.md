# Notes
Just some place for notes

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
