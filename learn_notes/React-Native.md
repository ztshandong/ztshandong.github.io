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

cmd+E 打开最近文件
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
# 组件生命周期
```jsx
//setup.js文件
import LifecycleComponent from './LifecycleComponent.js';
export default class setup extends Component {
    constructor(props){
        super(props);
        this.state=({
            remove:false //因为组件是在setup里加载的，所以只能在setup里移除
        })
    }
    render() {
        var view=this.state.remove?null:<LifecycleComponent name="张三"/>
        var text=this.state.remove?"复活":"干掉";
        return (
            <View style={styles.container}>
                <Text style={{backgroundColor:'yellow'}}
                onPress={()=>{
                    this.setState({
                        remove:!this.state.remove
                    })
                }}
                >{text}</Text>
                {view}
            </View>
        );
    }
}


//LifecycleComponent.js文件内容
//ES6
export default class LifecycleComponent extends Component {
    constructor(props) {
        console.log('----constructor----');
        super(props) //完成组件初始化，因为继承了Component，所以调用super，让父组件先初始化
        this.state = {
            count: 0,
        }
    }

    // getInitialState() {
    //     console.log('----getInitialState----');
    // }

    componentWillMount() {
        console.log('----componentWillMount----');
    }

    componentDidMount() {
        console.log('---componentDidMount-----');
    }

    componentWillReceiveProps(nextProps) {
        console.log('----componentWillReceiveProps----');
    }

    shouldComponentUpdate(nextProps, nextState) {
        console.log('----shouldComponentUpdate----');
        return true;
    }

    componentWillUpdate(nextProps, nextState) {
        console.log('----componentWillUpdate----');
        // 你不能在该方法中使用 this.setState()。如果需要更新 state 来响应某个 prop 的改变，请使用 componentWillReceiveProps
    }

    componentDidUpdate(prevProps, prevState) {
        console.log('----componentDidUpdate----');
    }

    componentWillUnmount() {
        console.log('----componentWillUnmount----');
    }

    render() {
        console.log('----render----');
        //return 与 <View> 必须在同一行，否则报错，无语
        return <View>
            <Text
                style={{backgroundColor: 'red'}}
                onPress={()=> {
                    this.setState({
                        count: this.state.count + 1,
                    })
                }}
            >hello {this.props.name}</Text>
            <Text>
                你点了我{this.state.count}次
            </Text>
        </View>
    }
}
```
# 导出组件
```jsx
//EIComponent.js
//ES6
export default class  EIComponent extends Component{
    render(){
     return <Text style={{backgroundColor:'red'}}>hello {this.props.name}</Text>
    }
}

var name='李四'; //变量
const age=18;  //常量
export {name,age};

export function sum(a,b) {
    return a+b;
}

//setup.js
import EIComponent,{name,age,sum} from './EIComponent.js';

export default class setup extends Component {
    constructor(props){
        super(props);
        this.state=({
            remove:false ,//因为组件是在setup里加载的，所以只能在setup里移除
            result:''
        })
    }
    render(){
        return (
            <View style={styles.container}>
                <Text style={{fontSize:50}}>{name}</Text>
                <Text style={{fontSize:50}}>{age}</Text>
                <Text style={{fontSize:50}}
                onPress={()=>{
                   var sumre= sum(2,3);
                    this.setState({
                        result:sumre
                    })
                }}
                >2+3={this.state.result}</Text>
                <Text style={{fontSize:50}}><EIComponent name="张三"/></Text>
                </View>);
    }

```
