# WebStorm
react-native init projectname

WebStorm的Terminal中即可运行

react-native run-ios   或  react-native run-android  (需要事先运行安卓模拟器)

cmd+1,2,3,4,5,6调整模拟器大小

打开模拟器自动更新，置顶功能，前端调试

⌘——Command

⌃ ——Control

⌥——alt

⇧——Shift

⇪——Caps Lock

fn——功能键就是fn

cmd+Alt+L格式化
# 定义组件，三种方式
```jsx

//ES6
export default class  HelloComponent extends Component{
    render(){
     return <Text style={{backgroundColor:'red'}}>hello {this.props.name}</Text>
        // return <Text style={{fontSize:20,backgroundColor:'red'}}>Hello</Text>
    }
}

//ES5
var HelloComponent = React.createClass({
    render(){
        return <Text style={{backgroundColor: 'green'}}>hello {this.props.name}</Text>
        // return <Text style={{fontSize:20,backgroundColor:'red'}}>Hello</Text>
    }
})
module.exports=HelloComponent;

//函数，无状态，不可使用this，但可以使用属性props
function HelloComponent(props) {
    return <Text style={{backgroundColor: 'blue'}}>hello {props.name}</Text>
}
module.exports=HelloComponent;


```
