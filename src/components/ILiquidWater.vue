<template>
    <div idm-ctrl="idm_module" :id="moduleObject.id" :idm-ctrl-id="moduleObject.id" class="ILiquidWater_app">
        <div v-if="propData.showTimeSearch" class="search_block">
            <a-month-picker @change="onChange" :defaultValue="moment('2022-01-01', 'YYYY-MM')"  valueFormat="YYYY-MM" placeholder="选择月份">
                <a-icon slot="suffixIcon" type="smile" />
            </a-month-picker>
        </div>
        <div class="table_block flex_between">
            <div v-if="propData.showStyle == '1' || propData.showStyle == '3'" class="table_list">
                <div id="myChartWhater1" class="table_box"></div>
                <div v-if="propData.showDescribe" class="describe">{{ propData.describeLeft }}</div>
            </div>
            <div v-if="propData.showStyle == '2' || propData.showStyle == '3'" class="table_list">
                <div id="myChartWhater2" class="table_box"></div>
                <div v-if="propData.showDescribe" class="describe">{{ propData.describeRight }}</div>
            </div>
        </div>
    </div>
</template>
  
<script>
import moment from 'moment';
import 'moment/locale/zh-cn';
import { getLiquidDataLeft,getLiquidDataRight } from '../mock/mockData.js'
import * as echarts from 'echarts'
import 'echarts-liquidfill/src/liquidFill.js'
export default {
    name: 'ILiquidWater',
    data() {
        return {
            moduleObject: {},
            propData: this.$root.propData.compositeAttr || {
                showTimeSearch: true,
                showStyle: '3',
                showLabelLeft: true,
                showLabelRight: true
            },
            form_time: '',
            value_left: 8000,
            value_right: 7000,
            maxDataLeft: 10000,
            maxDataRight: 10000,
            echarts_option1: {},
            echarts_option2: {},
            myChart1: null,
            myChart2: null,
            moment
        }
    },
    props: {
    },
    watch: {
        'propData.showStyle': function(value,old) {
            this.$nextTick(() => {
                this.initEcharts()
            })
        }
    },
    created() {
        this.moduleObject = this.$root.moduleObject;
        this.convertAttrToStyleObject();
        this.initEchartsOption()
    },
    mounted() {
        this.$nextTick(function (params) {
            this.initEcharts()
            this.getInitData()
        });
    },
    destroyed() { },
    methods: {
        getInitData() {
            if ( this.showStyle == 1 ) {
                this.getTableDataLeft()
            } else if ( this.showStyle == 2 ) {
                this.getTableDataRight()
            } else if ( this.showStyle == 3 ) {
                this.getTableDataLeft()
                this.getTableDataRight()
            }
        },
        getTableDataLeft() {
            if ( this.propData.dataSourceLeft && this.propData.dataSourceLeft.length ) {
                IDM.datasource.request(this.propData.dataSourceLeft[0].id,{
                    moduleObject:this.moduleObject,
                    param:{id:'获取过来的详情ID'}
                },function(res){
                    this.value_left = res[this.propData.dataFieldValueLeft || 'value'];

                    let total_left = res[this.propData.dataFieldTotalLeft || 'total'];
                    if ( total_left ) {
                        this.maxDataLeft = total_left;
                    }
                    this.updateEcharts()
                },function(error){
                    //这里是请求失败的返回结果
                })
            } else {
                this.value_left = getLiquidDataLeft().value;
                this.maxDataLeft = getLiquidDataLeft().total;
            }
        },
        getTableDataRight() {
            if ( this.propData.dataSourceLeft && this.propData.dataSourceLeft.length ) {
                IDM.datasource.request(this.propData.dataSourceLeft[0].id,{
                    moduleObject:this.moduleObject,
                    param:{id:'获取过来的详情ID'}
                },function(res){
                    this.value_right = res[this.propData.dataFieldValueRight || 'value'];
                    
                    let total_right = res[this.propData.dataFieldTotalRight || 'total']
                    if ( total_right ) {
                        this.maxDataRight = total_right;
                    }
                    this.updateEcharts()
                },function(error){
                    //这里是请求失败的返回结果
                })
            } else {
                this.value_left = getLiquidDataRight().value;
                this.maxDataLeft = getLiquidDataRight().total;
            }
        },
        getColorValue(element,default_value) {
            if (element && element.hex8) {
                return element.hex8
            } else {
                return default_value
            }
        },
        initEchartsOption() {
            let percent_left = parseInt( this.value_left / this.maxDataLeft * 100 ) / 100; 
            let percent_right = parseInt( this.value_right / this.maxDataRight * 100 ) / 100; 
            var option1 = {
                series: [
                    {
                        type: 'liquidFill',
                        data: [ percent_left ],
                        label: {
                            show: this.propData.showLabelLeft,
                            fontSize: this.propData.labelFontSizeLeft || 20,
                            color: this.getColorValue(this.propData.labelFontColorLeft,"#000"),
                            formatter: String(this.value_left)
                        },
                        radius: this.propData.radiusLeft || '80%',
                        color: [ this.getColorValue(this.propData.tableColorLeft,'red') ],
                        backgroundStyle: {
                            color: this.getColorValue(this.propData.tableBgColorLeft,'#ffffff')
                        },
                        outline: {
                            show: this.propData.showOutlineLeft, //是否显示轮廓 布尔值
                            borderDistance: this.propData.borderDistanceLeft || 0, //外部轮廓与图表的距离 数字
                            itemStyle: {
                                borderColor: this.getColorValue(this.propData.borderColorLeft,'blue'), //边框的颜色
                                borderWidth: this.propData.borderWidthLeft || 5 //边框的宽度
                            }
                        },
                    }
                ]
            };
            var option2 = {
                series: [
                    {
                        type: 'liquidFill',
                        data: [ percent_right ],
                        label: {
                            show: this.propData.showLabelRight,
                            fontSize: this.propData.labelFontSizeRight || 20,
                            color: this.getColorValue(this.propData.labelFontColorRight,"#000"),
                            formatter: String(this.value_right)
                        },
                        radius: this.propData.radiusRight || '80%',
                        color: [ this.getColorValue(this.propData.tableColorRight,'red') ],
                        backgroundStyle: {
                            color: this.getColorValue(this.propData.tableBgColorRight,'#ffffff')
                        },
                        outline: {
                            show: this.propData.showOutlineRight, //是否显示轮廓 布尔值
                            borderDistance: this.propData.borderDistanceRight || 0, //外部轮廓与图表的距离 数字
                            itemStyle: {
                                borderColor: this.getColorValue(this.propData.borderColorRight,'blue'), //边框的颜色
                                borderWidth: this.propData.borderWidthRight || 5 //边框的宽度
                            }
                        },
                    }
                ]
            };
            this.echarts_option1 = JSON.parse(JSON.stringify(option1))
            this.echarts_option2 = JSON.parse(JSON.stringify(option2))
        },
        onChange(e) {
            console.log(e)
            if ( e ) {
                this.form_time = e;
            } else {
                this.form_time = '';
            }
        },
        initEcharts() {
            let myChart1,myChart2;
            if ( this.propData.showStyle == 1 ) {
                myChart1 = echarts.init(document.getElementById("myChartWhater1"));
            } else if ( this.propData.showStyle == 2 ) {
                myChart2 = echarts.init(document.getElementById("myChartWhater2"));
            } else if ( this.propData.showStyle == 3 ) {
                myChart1 = echarts.init(document.getElementById("myChartWhater1"));
                myChart2 = echarts.init(document.getElementById("myChartWhater2"));
            } 
            this.myChart1 = myChart1;
            this.myChart2 = myChart2;
            if ( myChart1 ) {
                myChart1.setOption(this.echarts_option1);
            }
            if ( myChart2 ) {
                myChart2.setOption(this.echarts_option2);
            }
        },
        updateEcharts() {
            if ( this.myChart1 ) {
                console.log('图表刷新')
                this.myChart1.setOption(this.echarts_option1,true);
                this.myChart1.resize()
            }
            if ( this.myChart2 ) {
                this.myChart2.setOption(this.echarts_option2,true);
                this.myChart2.resize()
            }
        },
        getPercent(item) {
            let percent = parseInt(item.number / this.maxData * 100);
            return percent;
        },
        convertThemeListAttrToStyleObject() {
            const themeList = this.propData.themeList;
            if ( (!themeList) || !themeList.length ) {
                return
            }
            const themeNamePrefix = IDM.setting && IDM.setting.applications && IDM.setting.applications.themeNamePrefix ? IDM.setting.applications.themeNamePrefix : "idm-theme-";
            for (var i = 0; i < themeList.length; i++) {
                var item = themeList[i];
                
                if(item.key!=IDM.theme.getCurrentThemeInfo()){
                    //此处比对是不渲染输出不用的样式，如果页面会刷新就可以把此处放开
                    continue;
                }
                let backgroundObject = {
                    'background-color': item.mainColor ? item.mainColor.hex8 : "",
                }
                IDM.setStyleToPageHead( "." + themeNamePrefix + item.key + " #" + (this.moduleObject.packageid || "module_demo") + " .ISortTable_app_main .ant-progress-status-success .ant-progress-inner .ant-progress-bg", backgroundObject );
                IDM.setStyleToPageHead( "." + themeNamePrefix + item.key + " #" + (this.moduleObject.packageid || "module_demo") + " .ISortTable_app_main .ant-progress-inner .ant-progress-bg", backgroundObject );
            }
        },
        /** * 提供父级组件调用的刷新prop数据组件 */
        propDataWatchHandle(propData) {
            this.propData = propData.compositeAttr || {};
            this.convertAttrToStyleObject();
            this.initEchartsOption()
            this.$nextTick(() => {
                this.updateEcharts()
            })
        },
        /** * 把属性转换成样式对象 */
        convertAttrToStyleObjectTitle() {
            var styleObject = {};
            for (const key in this.propData) {
                if (this.propData.hasOwnProperty.call(this.propData, key)) {
                    const element = this.propData[key];
                    if (!element && element !== false && element != 0) {
                        continue;
                    }
                    switch (key) {
                        case "heightTitle":
                            styleObject['height'] = element;
                            styleObject['line-height'] = element;
                            break;
                        case 'fontTitle':
                            IDM.style.setFontStyle(styleObject,element);
                            break;
                        case 'boxTitle':
                            IDM.style.setBoxStyle(styleObject,element);
                            break;
                    }
                }
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_title', styleObject);
        },
        convertAttrToStyleObjectDescribe() {
            var styleObject = {};
            for (const key in this.propData) {
                if (this.propData.hasOwnProperty.call(this.propData, key)) {
                    const element = this.propData[key];
                    if (!element && element !== false && element != 0) {
                        continue;
                    }
                    switch (key) {
                        case "heightDescribe":
                            styleObject['height'] = element;
                            styleObject['line-height'] = element;
                            break;
                        case 'fontDescribe':
                            IDM.style.setFontStyle(styleObject,element);
                            break;
                        case 'boxDescribe':
                            IDM.style.setBoxStyle(styleObject,element);
                            break;
                    }
                }
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_footer', styleObject);
        },
        convertAttrToStyleObjectList() {
            var styleObject = {};
            var styleObjectName = {};
            var styleObjectNumber = {};
            var styleObjectProcess = {};
            var styleObjectProcessBg = {};
            var styleObjectIndex = {};
            for (const key in this.propData) {
                if (this.propData.hasOwnProperty.call(this.propData, key)) {
                    const element = this.propData[key];
                    if (!element && element !== false && element != 0) {
                        continue;
                    }
                    switch (key) {
                        case "heightDescribe":
                            styleObject['height'] = element;
                            break;
                        case 'widthListName':
                            styleObjectName['width'] = element;
                            break;
                        case 'widthListNumber':
                            styleObjectNumber['width'] = element;
                            break;
                        case 'fontListName':
                            IDM.style.setFontStyle(styleObjectName,element)
                            break;
                        case 'fontListNumber':
                            IDM.style.setFontStyle(styleObjectNumber,element)
                            break;
                        case 'boxList':
                            IDM.style.setBoxStyle(styleObject,element);
                            break;
                        case 'processColor':
                            if (element && element.hex8) {
                                styleObjectProcess["background-color"] = element.hex8 + ' !important';
                            }
                            break;
                        case 'processBgColor':
                            if (element && element.hex8) {
                                styleObjectProcessBg["background-color"] = element.hex8;
                            }
                            break;
                        case 'widthListIndex':
                            styleObjectIndex['width'] = element;
                            break;
                        case 'fontListIndex':
                            IDM.style.setBoxStyle(styleObjectIndex,element);
                            break;
                    }
                }
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_main .list .index', styleObjectIndex);
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_main .list .name', styleObjectName);
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_main .list .number', styleObjectNumber);
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_main .ant-progress-status-success .ant-progress-inner .ant-progress-bg', styleObjectProcess);
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_main .ant-progress-inner .ant-progress-bg', styleObjectProcess);
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_main .ant-progress-inner', styleObjectProcessBg);
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .ISortTable_app_main .list', styleObject);
        },
        convertAttrToStyleObject() {
            this.convertAttrToStyleObjectTitle()
            this.convertAttrToStyleObjectDescribe()
            this.convertAttrToStyleObjectList()
            this.convertThemeListAttrToStyleObject()
            var styleObject = {};
            if (this.propData.bgSize && this.propData.bgSize == "custom") {
                styleObject["background-size"] = (this.propData.bgSizeWidth ? this.propData.bgSizeWidth.inputVal + this.propData.bgSizeWidth.selectVal : "auto") + " " + (this.propData.bgSizeHeight ? this.propData.bgSizeHeight.inputVal + this.propData.bgSizeHeight.selectVal : "auto")
            } else if (this.propData.bgSize) {
                styleObject["background-size"] = this.propData.bgSize;
            }
            if (this.propData.positionX && this.propData.positionX.inputVal) {
                styleObject["background-position-x"] = this.propData.positionX.inputVal + this.propData.positionX.selectVal;
            }
            if (this.propData.positionY && this.propData.positionY.inputVal) {
                styleObject["background-position-y"] = this.propData.positionY.inputVal + this.propData.positionY.selectVal;
            }
            for (const key in this.propData) {
                if (this.propData.hasOwnProperty.call(this.propData, key)) {
                    const element = this.propData[key];
                    if (!element && element !== false && element != 0) {
                        continue;
                    }
                    switch (key) {
                        case "width":
                        case "height":
                            styleObject[key] = element;
                            break;
                        case "bgColor":
                            if (element && element.hex8) {
                                styleObject["background-color"] = element.hex8;
                            }
                            break;
                        case "box":
                            IDM.style.setBoxStyle(styleObject,element)
                            break;
                        case "bgImgUrl":
                            styleObject["background-image"] = `url(${window.IDM.url.getWebPath(element)})`;
                            break;
                        case "positionX":
                            //背景横向偏移

                            break;
                        case "positionY":
                            //背景纵向偏移

                            break;
                        case "bgRepeat":
                            //平铺模式
                            styleObject["background-repeat"] = element;
                            break;
                        case "bgAttachment":
                            //背景模式
                            styleObject["background-attachment"] = element;
                            break;
                        case "border":
                            IDM.style.setBorderStyle(styleObject,element)
                            break;
                        case "font":
                            IDM.setFontStyle(styleObject,element)
                            break;
                    }
                }
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id, styleObject);
        },
        /**
         * 通用的url参数对象
         * 所有地址的url参数转换
         */
        commonParam() {
            let urlObject = IDM.url.queryObject();
            var params = {
                pageId:
                    window.IDM.broadcast && window.IDM.broadcast.pageModule
                        ? window.IDM.broadcast.pageModule.id
                        : "",
                urlData: JSON.stringify(urlObject),
            };
            return params;
        },
        /**
         * 重新加载
         */
        reload() {
            //请求数据源

        },
        
        /**
         * 通用的获取表达式匹配后的结果
         */
        getExpressData(dataName, dataFiled, resultData) {
            //给defaultValue设置dataFiled的值
            var _defaultVal = undefined;
            if (dataFiled) {
                var filedExp = dataFiled;
                filedExp =
                    dataName +
                    (filedExp.startsWiths("[") ? "" : ".") +
                    filedExp;
                var dataObject = { IDM: window.IDM };
                dataObject[dataName] = resultData;
                _defaultVal = window.IDM.express.replace.call(
                    this,
                    "@[" + filedExp + "]",
                    dataObject
                );
            }
            //对结果进行再次函数自定义
            if (this.propData.customFunction && this.propData.customFunction.length > 0) {
                var params = this.commonParam();
                var resValue = "";
                try {
                    resValue = window[this.propData.customFunction[0].name] && window[this.propData.customFunction[0].name].call(this, {
                        ...params,
                        ...this.propData.customFunction[0].param,
                        moduleObject: this.moduleObject,
                        expressData: _defaultVal, interfaceData: resultData
                    });
                } catch (error) {
                }
                _defaultVal = resValue;
            }

            return _defaultVal;
        },
        /**
         * 组件通信：接收消息的方法
         * @param {
         *  type:"发送消息的时候定义的类型，这里可以自己用来要具体做什么，统一规定的type：linkageResult（组件联动传结果值）、linkageDemand（组件联动传需求值）、linkageReload（联动组件重新加载）
         * 、linkageOpenDialog（打开弹窗）、linkageCloseDialog（关闭弹窗）、linkageShowModule（显示组件）、linkageHideModule（隐藏组件）、linkageResetDefaultValue（重置默认值）"
         *  message:{发送的时候传输的消息对象数据}
         *  messageKey:"消息数据的key值，代表数据类型是什么，常用于表单交互上，比如通过这个key判断是什么数据"
         *  isAcross:如果为true则代表发送来源是其他页面的组件，默认为false
         * } object 
         */
        receiveBroadcastMessage(object) {
            console.log("组件收到消息", object)
        },
        /**
         * 组件通信：发送消息的方法
         * @param {
         *  type:"自己定义的，统一规定的type：linkageResult（组件联动传结果值）、linkageDemand（组件联动传需求值）、linkageReload（联动组件重新加载）
         * 、linkageOpenDialog（打开弹窗）、linkageCloseDialog（关闭弹窗）、linkageShowModule（显示组件）、linkageHideModule（隐藏组件）、linkageResetDefaultValue（重置默认值）",
         *  message:{实际的消息对象},
         *  rangeModule:"为空发送给全部，根据配置的属性中设定的值（值的内容是组件的packageid数组），不取子表的，比如直接 this.$root.propData.compositeAttr["attrKey"]（注意attrKey是属性中定义的bindKey）,这里的格式为：['1','2']"",
         *  className:"指定的组件类型，比如只给待办组件发送，然后再去过滤上面的值"
         *  globalSend:如果为true则全站发送消息，注意全站rangeModule是无效的，只有className才有效，默认为false
         * } object 
         */
        sendBroadcastMessage(object) {
            window.IDM.broadcast && window.IDM.broadcast.send(object);
        },
        /**
         * 交互功能：设置组件的上下文内容值
         * @param {
         *  type:"定义的类型，已知类型：pageCommonInterface（页面统一接口返回值）、"
         *  key:"数据key标识，页面每个接口设置的数据集名称，方便识别获取自己需要的数据"
         *  data:"数据集，内容为：字符串 or 数组 or 对象"
         * }
         */
        setContextValue(object) {
            console.log("统一接口设置的值", object);
            if (object.type != "pageCommonInterface") {
                return;
            }
            //这里使用的是子表，所以要循环匹配所有子表的属性然后再去设置修改默认值
            if (object.key == this.propData.dataName) {
                // this.propData.fontContent = this.getExpressData(this.propData.dataName,this.propData.dataFiled,object.data);
                this.$set(this.propData, "fontContent", this.getExpressData(this.propData.dataName, this.propData.dataFiled, object.data));
            }
        }
    }
}
</script>
<style lang="scss" scoped>
.ILiquidWater_app{
    .table_box{
        width: 50%;
        min-width: 120px;
        min-height: 120px;
    }
}
</style>
<style lang="scss">

</style>>