<template>
    <div class="address-wrap">
        <div class="tab-menu">
            <div @click.stop="tabActive(1)" :class="{active: tabIndex == 1}">省份</div>
            <div @click.stop="tabActive(2)" :class="{active: tabIndex == 2}">城市</div>
            <div @click.stop="tabActive(3)" :class="{active: tabIndex == 3}">县区</div>
        </div>
        <div class="tab-main">
            <div class="province-wrap" v-if="operType == 1">
                <div class="row-1 row">
                    <div class="name-range">A-G</div>
                    <div class="item-block">
                        <div @click.stop="selId(item)" :class="{active: objectId == item.objectId}" class="item" v-for="(item, index) in row1">{{item.objectName}}</div>
                    </div>
                </div>
                <div class="row-2 row">
                    <div class="name-range">H-K</div>
                    <div class="item-block">
                        <div @click.stop="selId(item)" :class="{active: objectId == item.objectId}" class="item" v-for="(item, index) in row2">{{item.objectName}}</div>
                    </div>
                </div>
                <div class="row-3 row">
                    <div class="name-range">L-S</div>
                    <div class="item-block">
                        <div @click.stop="selId(item)" :class="{active: objectId == item.objectId}" class="item" v-for="(item, index) in row3">{{item.objectName}}</div>
                    </div>
                </div>
                <div class="row-4 row">
                    <div class="name-range">T-Z</div>
                    <div class="item-block">
                        <div @click.stop="selId(item)" :class="{active: objectId == item.objectId}" class="item" v-for="(item, index) in row4">{{item.objectName}}</div>
                    </div>
                </div>
            </div>
            <div class="city-wrap" v-else-if="operType == 2">
                <div @click.stop="selId1(item)" :class="{active: cityId == item.objectId}" class="item" v-for="(item, index) in cityList">{{item.objectName}}</div>
            </div>
            <div class="area-wrap" v-else-if="operType == 3">
                <div @click.stop="complateId(item)" :class="{active: areaId == item.objectId}" class="item" v-for="(item, index) in areaList">{{item.objectName}}</div>
            </div>
        </div>
    </div>
</template>

<script>
// const pinyinTable = require('./pinyin')
import {pinyinTable} from './pinyin'
import {seekGetAddress} from 'Api/commonality/seek'
export default {
    data () {
        return {
            tabIndex: 1,
            provinceList: [],
            cityList: [],
            areaList: [],
            row1: [],
            row2: [],
            row3: [],
            row4: [],
            objectId: '',
            objectName: '',
            cityId: '',
            cityName: '',
            areaId: '',
            areaName: '',
            operType: 1

        }
    },
    created () {
        // console.log(this.ucfirst('北京'))
        // console.log(this.ConvertPinyin('北京'))
        this.getAddress()
    },
    methods:{
        selId (item) { // 选择省id
            console.log(item)
            this.objectId = item.objectId
            this.objectName = item.objectName
            this.operType = 2
            this.tabIndex = 2
            this.getCityAddress()
        },
        selId1 (item) { // 选择省id
            console.log(item)
            this.cityId = item.objectId
            this.cityName = item.objectName
            this.operType = 3
            this.tabIndex = 3
            this.getAreaAddress()
        },
        complateId (item) {
            this.areaId = item.objectId
            this.areaName = item.objectName
            this.$emit('addressReturn', {
                provId: this.objectId,
                provName: this.objectName,
                cityId: this.cityId,
                cityName: this.cityName,
                areaId: this.areaId,
                areaName: this.areaName
                })
        },
        tabActive (index) { // 切换省市区列表
            this.tabIndex = index
            this.operType = index 
        },
        addressSort (list) {
            this.row1 = []
            this.row2 = []
            this.row3 = []
            this.row4 = []
            list.forEach((item, index) => {
                let W = this.ConvertPinyin(item.objectName).substring(0,1)
                Object.assign(item, {
                    initials: W
                })
                if (W.charCodeAt() >= 65 && W.charCodeAt() <= 71) {
                    this.row1.push(item)
                } else if (W.charCodeAt() >= 72 && W.charCodeAt() <= 75) {
                    this.row2.push(item)
                } else if (W.charCodeAt() >= 76 && W.charCodeAt() <= 83) {
                    this.row3.push(item)
                } else if (W.charCodeAt() >= 84 && W.charCodeAt() <= 90) {
                    this.row4.push(item)
                }
            })
        },
        getAddress () { // 获取省的地址
            let options = {
                type: this.operType,
            }
            seekGetAddress(options).then((res) => {
                console.log(res)
                if (res.data.state == 200) {
                    this.provinceList = res.data.data.dataList
                    this.addressSort(this.provinceList)
                }
            }, (res) => {

            })
        },
        getCityAddress () { // 获取市的地址
            let options = {
                type: this.operType,
                objectId: this.objectId
            }
            seekGetAddress(options).then((res) => {
                console.log(res)
                if (res.data.state == 200) {
                    this.cityList = res.data.data.dataList
                    //this.addressSort(this.cityList)
                }
            }, (res) => {

            })
        },
        getAreaAddress () { // 获取区的地址
            let options = {
                type: this.operType,
                objectId: this.cityId
            }
            seekGetAddress(options).then((res) => {
                console.log(res)
                if (res.data.state == 200) {
                    this.areaList = res.data.data.dataList
                    //this.addressSort(this.areaList)
                }
            }, (res) => {

            })
        },
        // 汉字转拼音  
        ConvertPinyin(l1) {  
            var l2 = l1.length;  
            var I1 = "";  
            var reg = new RegExp('[a-zA-Z0-9\- ]');  
            for (var i = 0; i < l2; i++) {  
                var val = l1.substr(i, 1);  
                var name = this.arraySearch(val, pinyinTable);  
                if (reg.test(val)) {  
                    I1 += val;  
                } else if (name !== false) {  
                    I1 += name;  
                }  

            }  
            I1 = I1.replace(/ /g, '-');  
            while (I1.indexOf('--') > 0) {  
                I1 = I1.replace('--', '-');  
            }  
            return I1;  
        },
        // 在对象中搜索  
        arraySearch(l1, l2) {  
            for (var name in pinyinTable) {  
                if (pinyinTable[name].indexOf(l1) != -1) {  
                    return this.ucfirst(name); break;  
                }  
            }  
            return false;  
        },
        // 首字母大写  
        ucfirst(l1) {  
            if (l1.length > 0) {  
                var first = l1.substr(0, 1).toUpperCase();  
                var spare = l1.substr(1, l1.length);  
            return first + spare;  
            //return first;  
            }  
        }  

    }
}
</script>

<style lang="scss" scoped>
.address-wrap {
    width: 301px;
    height: 240px;
    border: 1px solid #d6d6d6;
    border-radius: 4px;
    position: absolute;
    top: 30px;
    left: 50px;
    background:#fff;
    z-index: 100;
    .tab-menu {
        height: 28px;
        width: 100%;
        background:#d6d6d6;
        
        &>div {
            width: 99px;
            height: 100%;
            float: left;
            background:#f6f7f8;
            font-size: 14px;
            color:#333333;
            text-align: center;
            line-height: 28px;
            border-bottom: 1px solid #d6d6d6;
            cursor: pointer;
        }
        &>div:nth-child(1), &>div:nth-child(2) {
            margin-right:1px;
        }
        .active {
            background:#fff;
            border-bottom: none;
            color:#2993f8;
            
        }
    }
    .tab-main {
        padding-top:6px;
        .province-wrap {
            .row {
                width: 100%;
                height: 48px;
                .name-range {
                    width: 42px;
                    height: 100%;
                    float: left;
                    font-size: 12px;
                    font-weight: bold;
                    color:#2993f8;
                    line-height: 24px;
                    padding-left: 12px;
                }
                .item-block {
                    width: 256px;
                    height: 100%;
                    float: left;
                    .item {
                        width: 48px;
                        height: 24px;
                        // background:#e9f4fe;
                        float: left;
                        border-radius: 2px;
                        color:#333;
                        font-size: 12px;
                        text-align:center;
                        line-height: 24px;
                        white-space: nowrap;
                        text-overflow: ellipsis;
                        overflow: hidden;
                        cursor: pointer;
                    }
                    .item.active {
                        background:#e9f4fe;
                        color:#2993f8;
                    }
                    .item:hover {
                        background:#e9f4fe;
                        color:#2993f8;
                    }
                }
            }
        }
        .city-wrap {
            padding: 0 25px;
            .item {
                width: 48px;
                height: 24px;
                // background:#e9f4fe;
                float: left;
                border-radius: 2px;
                color:#333;
                font-size: 12px;
                text-align:center;
                line-height: 24px;
                white-space: nowrap;
                text-overflow: ellipsis;
                overflow: hidden;
                cursor: pointer;
            }
            .item:hover {
                background:#e9f4fe;
                color:#2993f8;
            }
            .item.active {
                background:#e9f4fe;
                color:#2993f8;
            }
        }
        .area-wrap {
            padding: 0 25px;
            .item {
                width: 48px;
                height: 24px;
                // background:#e9f4fe;
                float: left;
                border-radius: 2px;
                color:#333;
                font-size: 12px;
                text-align:center;
                line-height: 24px;
                white-space: nowrap;
                text-overflow: ellipsis;
                overflow: hidden;
                cursor: pointer;
            }
            .item:hover {
                background:#e9f4fe;
                color:#2993f8;
            }
            .item.active {
                background:#e9f4fe;
                color:#2993f8;
            }
        }
        
    }
}
</style>
