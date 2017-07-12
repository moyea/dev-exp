## 访问器(Accessors)
TypeScript支持利用getters/setters来控制对成员的访问。让我们可以控制类的成员之间的访问方式。
```typescript
class Person{
  private _name: string;
  constructor(){
  }
  set name(value: string){
    this._name = value;
  }
  get name(){
    return this._name;
  }
}
const person = new Person();
person._name = 'Tom';// error:私有属性无法被外部访问
person.name='Tom'; // 调用setter来设置私有的_name
console.log(person.name); // 调用getter来获取私有的_name
```
被private修饰的属性为私有属性,但是可以通过getters/setters来获取值
