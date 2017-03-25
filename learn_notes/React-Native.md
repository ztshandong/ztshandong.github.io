# WebStorm
react-native init proname
WebStorm的Terminal中即可运行
react-native run-ios
cmd+1,2,3,4,5,6调整模拟器大小
打开模拟器自动更新，前段调试，置顶功能
cmd+Alt+L格式化
# 定义组件
```jsx

//ES6
export default class  HelloComponent extends Component{
    render(){
     return <Text style={{backgroundColor:'red'}}>hello</Text>
        // return <Text style={{fontSize:20,backgroundColor:'red'}}>Hello</Text>
    }
}

//ES5
var HelloComponent = React.createClass({
    render(){
        return <Text style={{backgroundColor: 'green'}}>hello</Text>
        // return <Text style={{fontSize:20,backgroundColor:'red'}}>Hello</Text>
    }
})
module.exports=HelloComponent;

//函数式
function HelloComponent() {
    return <Text style={{backgroundColor: 'blue'}}>hello</Text>
}
module.exports=HelloComponent;

```
