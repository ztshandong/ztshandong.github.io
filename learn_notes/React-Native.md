# WebStorm
- react-native init projectname
- WebStorm的Terminal中即可运行
- react-native run-ios   或  react-native run-android  (需要事先运行安卓模拟器)
- ⌘——Command
- ⌃——Control
- ⌥——alt
- ⇧——Shift
- ⇪——Caps Lock
- fn——功能键就是fn
 #
 # 快捷键
- ⌘+D 打开IOS模拟器调试菜单
- ⌘+M 打开Android模拟器调试菜单
- ⌘+1,2,3,4,5,6调整模拟器大小
- ⌘+Alt+L格式化
- ⌘+E 打开最近文件
- ⌘+⇧+u 切换所选字符大小写

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
# 导入组件
```jsx
import * as Page from './Page'
使用* as 修饰后，导入的组件直接就成为一个对象，可以使用Page.来调用方法及属性
```
# 特定平台
```sh
React Native会检测某个文件是否具有.ios.或是.android.的扩展名，创建两个文件
BigButton.ios.js
BigButton.android.js
import BigButton from './components/BigButton';即可
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
}
```
# Props，只读，不可修改
```jsx
//PropsTest.js
import React, {Component,PropTypes} from 'react';
import {
    // AppRegistry,
    StyleSheet,
    Text,
    View
} from 'react-native';

//ES6
export default class  PropsTest extends Component{
    static  defaultProps={
    name:'小明',
        age:16,

}
static propTypes={
    name:PropTypes.string,
    age:PropTypes.number,
    sex:PropTypes.string.isRequired
}
    render(){
     return <View>

        <Text style={{backgroundColor:'red'}}>hello {this.props.name}</Text>
        <Text style={{backgroundColor:'red'}}>hello {this.props.age}</Text>
        <Text style={{backgroundColor:'red'}}>hello {this.props.sex}</Text>
    </View>
    }
}

//setup.js
import PropsTest from './PropsTest.js';
export default class setup extends Component {
    constructor(props) {
        super(props);
        this.state = ({
            remove: false,//因为组件是在setup里加载的，所以只能在setup里移除
            result: ''
        })
    }

    render() {
        var params = {name: 'lisi', age: 38, sex: '男'}
        //var {name,sex}=params  //解构赋值，获取指定属性
        return (
            <View style={styles.container}>
                <PropsTest
                    {...params} //ES6延展操作符

                    //name={name}  //对应var {name,sex}=params，解构赋值
                    //sex={sex}

                    //name={params.name} //最复杂的代码
                    //age={params.age}
                    //sex={params.sex}
                />
            </View>
        );
    }
}
```
# State
```jsx
//setup.js
import StateTest from './StateTest.js';
export default class setup extends Component {
    constructor(props) {
        super(props);
        this.state = ({
            remove: false,//因为组件是在setup里加载的，所以只能在setup里移除
            result: ''
        })
    }
render() {
    return (
        <View style={styles.container}>
            <StateTest
            />
        </View>
    );
}


//StateTest.js
import React, {Component} from 'react';
import {
    // AppRegistry,
    StyleSheet,
    Text,
    View,
    Image
} from 'react-native';

//ES6
export default class StateTest extends Component {
    //方法一：
    state = {
        size: 80,
    }
    //state是私有的，无法通过其他组件传递，要在constructor构造方法中初始化
    constructor(props) {
        super(props);
        //方法二
        // this.state={
        //     size:80,
        // }
    }

    render() {
        return <View>
            <Text
                onPress={()=> {
                    this.setState({
                        size: this.state.size + 10
                    })
                }}
            >长大</Text>
            <Text
                onPress={()=> {
                    this.setState({
                        size: this.state.size - 10
                    })
                }}
            >变小</Text>
            <Image
                style={{width: this.state.size, height: this.state.size}}
                source={require('./kid.jpg')}
            ></Image>
        </View>
    }

    /*
     render() {
     return <Text style={{backgroundColor: 'red'}}>hello {this.state.size}</Text>
     // return <Text style={{fontSize:20,backgroundColor:'red'}}>Hello</Text>
     }
     */
}

```
# ref
```jsx
//RefTest.js
//ES6
export default class RefTest extends Component {
    //方法一：
    state = {
        size: 80,
    }
    //state是私有的，无法通过其他组件传递，要在constructor构造方法中初始化
    constructor(props) {
        super(props);
        //方法二
        // this.state={
        //     size:80,
        // }
    }

    getSize() {
        return this.state.size;
    }

    render() {
        return <View>
            <Text
                onPress={()=> {
                    this.setState({
                        size: this.state.size + 10
                    })
                }}
            >长大</Text>
            <Text
                onPress={()=> {
                    this.setState({
                        size: this.state.size - 10
                    })
                }}
            >变小</Text>
            <Image
                style={{width: this.state.size, height: this.state.size}}
                source={require('./kid.jpg')}
            ></Image>
        </View>
    }

    /*
     render() {
     return <Text style={{backgroundColor: 'red'}}>hello {this.state.size}</Text>
     // return <Text style={{fontSize:20,backgroundColor:'red'}}>Hello</Text>
     }
     */
}




//setup.js
import RefTest from './RefTest.js';

export default class setup extends Component {
    constructor(props) {
        super(props);
        this.state = ({
            remove: false,//因为组件是在setup里加载的，所以只能在setup里移除
            result: '',
            size: 0
        })
    }

    //ref是内置属性
    render() {
        return (
            <View style={styles.container}>
                <Text
                    ref="ref2"
                    onPress={()=> {
                        var gsize = this.refs.reftest.getSize();
                        var gref2 = this.refs['ref2'];
                        //var gsize=this.reftest.getSize();//方式二，与下面对应
                        this.setState({
                            size: gsize,
                        })
                    }}
                >获取大小{this.state.size}</Text>
                <Text>{this.gref2}</Text>

                <RefTest
                    ref="reftest"
                    //ref={reftest=>this.reftest=reftest}//方式二
                />
            </View>
        );
    }
}
```
# class
```jsx
//Mi.js
import Student from './Student'
export default  class Mi extends Student{
    constructor(name,sex,age){
        super(name,sex,age);
    }
    getDiscription(){
        return '大家好，'+super.getDiscription();
    }
}


//Student.js
export  default  class Student {
    constructor(name, sex, age) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }

    getDiscription() {
        return '姓名:' + this.name + '，性别：' + this.sex + '，age:' + this.age
    }
}


//setup.js
import RefTest from './RefTest.js';
import Student from './Student.js';
import Mi from './Mi.js';

export default class setup extends Component {
    constructor(props) {
        super(props);
        this.state = ({
            remove: false,//因为组件是在setup里加载的，所以只能在setup里移除
            result: '',
            size: 0
        })
        this.stu = new Student('小米', '男', 8)
        this.mi = new Mi('睿睿','男',8);
    }

    //ref是内置属性
    render() {
        return (
            <View style={styles.container}>
                <Text
                    ref="ref2"
                    onPress={()=> {
                        var gsize = this.refs.reftest.getSize();
                        var gref2 = this.refs['ref2'];
                        //var gsize=this.reftest.getSize();//方式二，与下面对应
                        this.setState({
                            size: gsize,
                        })
                    }}
                >获取大小{this.state.size}</Text>
                <Text>{this.gref2}</Text>
                <Text>{this.stu.getDiscription()}</Text>
                <Text>{this.mi.getDiscription()}</Text>

                <RefTest
                    ref="reftest"
                    //ref={reftest=>this.reftest=reftest}//方式二
                />
            </View>
        );
    }
 }
```
# 原生图片
```sh
IOS，图片放入Images.xcassets文件夹下
Android，放入app/src/main/res/drawable-xxhdpi文件夹下
source={{uri: 'mikid'}}即可访问，不能加扩展名
```
