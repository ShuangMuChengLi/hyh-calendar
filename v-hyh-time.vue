<template>
    <div class="hyh-datetime-plugin-wrapper" @click.stop>
        <input v-model="inputValue" class="hyh-time-input" readonly @click.stop="inputValueClick"/>
        <div class="hyh-datetime-pop-wrapper" v-show="popShow">
            <ul class="hyh-datetime-list-ul">
                <li class="hyh-datetime-list-li "
                    @click.stop="changeTable('realTime')"
                    :class="{active:currentType === 'realTime'}">
                    <span>实时</span>
                    <span v-show="currentType === 'realTime'" class="hyh-datetime-list-li-value" >{{typeListValue}}</span>
                </li>
                <li class="hyh-datetime-list-li"
                    @click.stop="changeTable('current1')"
                    :class="{active:currentType === 'current1'}">
                    <span>最近1天</span>
                    <span v-show="currentType === 'current1'" class="hyh-datetime-list-li-value" >{{typeListValue}}</span>
                </li>
                <li class="hyh-datetime-list-li"
                    @click.stop="changeTable('current7')"
                    :class="{active:currentType === 'current7'}">
                    <span>最近7天</span>
                    <span v-show="currentType === 'current7'" class="hyh-datetime-list-li-value" >{{typeListValue}}</span>
                </li>
                <li class="hyh-datetime-list-li"
                    @click.stop="changeTable('current30')"
                    :class="{active:currentType === 'current30'}">
                    <span>最近30天</span>
                    <span v-show="currentType === 'current30'" class="hyh-datetime-list-li-value" >{{typeListValue}}</span>
                </li>
                <li class="hyh-datetime-list-li"
                    :class="{active:currentType === 'day'}"
                    @click.stop="changeTable('day')">日</li>
                <li class="hyh-datetime-list-li"
                    :class="{active:currentType === 'week'}"
                    @click.stop="changeTable('week')">周</li>
                <li class="hyh-datetime-list-li"
                    :class="{active:currentType === 'month'}"
                    @click.stop="changeTable('month')">月</li>
            </ul>
            <div class="hyh-datetime-hierarchy-wrapper" v-show="dayHierarchyShow || monthHierarchyShow">
                <div class="hyh-hierarchy-wrapper"  v-show="dayHierarchyShow">
                    <header class="hyh-hierarchy-toolbar">
                        <div class="hyh-hierarchy-selectleft-wrap noselect"
                            @click.stop="gotoPreMonth"><</div>
                        <div class="hyh-hierarchy-selectright-wrap"
                             @click.stop="gotoNextMonth">></div>
                        <div class="hyh-hierarchy-selectcenter-wrap noselect">
                            {{ViewTitle()}}
                        </div>
                    </header>
                    <div class="hyh-hierarchy-content-wrap">
                        <header class="hyh-hierarchy-calendar-header clearfix">
                            <span class="hyh-calendar-header-span">一</span>
                            <span class="hyh-calendar-header-span">二</span>
                            <span class="hyh-calendar-header-span">三</span>
                            <span class="hyh-calendar-header-span">四</span>
                            <span class="hyh-calendar-header-span">五</span>
                            <span class="hyh-calendar-header-span">六</span>
                            <span class="hyh-calendar-header-span">七</span>
                        </header>
                        <div class="hyh-hierarchy-calendar-content clearfix">
                            <span class=" not-current-view-item"
                                  :class="{
                                         active: dayActive(item),
                                         over:dayInWeekActive(item)
                                     }"
                                  @click.stop="selectDay(item)"
                                  v-for="item in preMonthDays">{{item.date()}}</span>
                            <span class="hyh-calendar-content-span"
                                  :class="{
                                         active: dayActive(item),
                                         over:dayInWeekActive(item)
                                     }"
                                  v-for="item in currentMonthDays"
                                  @click.stop="selectDay(item)">{{item.date()}}</span>
                            <span class=" not-current-view-item"
                                  :class="{
                                         active: dayActive(item),
                                         over:dayInWeekActive(item)
                                     }"
                                  @click.stop="selectDay(item)"
                                  v-for="item in nextMonthDays">{{item.date()}}</span>
                        </div>
                    </div>
                </div>
                <div class="hyh-hierarchy-wrapper"  v-show="monthHierarchyShow">
                    <header class="hyh-hierarchy-toolbar">
                        <div class="hyh-hierarchy-selectleft-wrap"
                            @click.stop = "gotoPreYear()"><</div>
                        <div class="hyh-hierarchy-selectright-wrap"
                             @click.stop = "gotoNextYear()">></div>
                        <div class="hyh-hierarchy-selectcenter-wrap">
                            {{currentViewYear}}年{{monthViewCurrentMonth ? monthViewCurrentMonth.month() + 1 + "月" : ""}}
                        </div>
                    </header>
                    <div class="hyh-hierarchy-content-wrap">
                        <div>
                            <div class="hyh-month-content-span"
                                 :class="{active: monthViewCurrentMonth && (i === monthViewCurrentMonth.month() + 1)}"
                                 @click.stop="selectMonth(i)"
                                 v-for="i in 12"
                            >{{i}}月</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    let moment = require("moment");
    let _ = require("lodash");
    export default {
        props:[
            "value",

        ],
        created() {

        },
        mounted() {
            document.addEventListener("click",()=>{
                this.popShow = false;
            });
            this.changeTable("realTime");
        },
        data() {
            return {
                popShow:false,
                currentType:"",
                inputValue:null,
                typeListValue:"",

                dayHierarchyShow:false,
                weekHierarchyShow:false,
                monthHierarchyShow:false,

                currentViewMonth:null,
                currentViewYear:null,


                dayViewCurrentDay: null,// 当前选中的日期
                monthViewCurrentMonth: null,// 当前选中的月份

                preMonthDays: [],
                currentMonthDays: [],
                nextMonthDays: [],


                weekFirstDateOfMonth: null,
                weekLastDateOfMonth: null
            }
        },
        computed: {

        },
        methods:{
            inputValueClick(){
                this.popShow = !this.popShow;
            },
            reset(){
                this.inputValue = null;

                this.currentViewMonth = null;
                this.currentViewYear = null;

                this.dayViewCurrentDay = null;
                this.monthViewCurrentMonth = null;

                this.preMonthDays = [];
                this.currentMonthDays = [];
                this.nextMonthDays = [];

                this.weekFirstDateOfMonth = null;
                this.weekLastDateOfMonth = null;
                this.$emit("input", {
                    type:null
                })
            },
            init(){
                this.reset();
                this.currentViewMonth = moment();
                this.currentViewYear = this.currentViewMonth.year();
                switch (this.currentType){
                    case "day":
                        this.buildCalendar();
                        break;
                    case "week":
                        this.buildCalendar();
                        break;
                    case "month":
                        break;
                }
            },
            // 获取上个月份一共多少天
            getMonthLastDays(obj){
                let year = obj.year;
                let m = moment().year(year).month(obj.month);
                let daysInMonth = m.daysInMonth();
                return daysInMonth;
            },
            // 获取月份第一天是星期几
            getMonthFirstDay(date){
                let firstDay = date.set("date", 1);
                return firstDay.day();
            },
            // 获取月份最后一天是星期几
            getMonthlastDay(date){
                let firstDay = date.set("date", date.daysInMonth());
                return firstDay.day();
            },
            buildCalendar(){
                let currentMonthObj = {
                    month:this.currentViewMonth.month(),
                    year : this.currentViewMonth.year()
                };
                let preMonthObj = {
                    month:currentMonthObj.month === 0 ? 11 : currentMonthObj.month - 1,
                    year : currentMonthObj.month === 0 ? currentMonthObj.year - 1 : currentMonthObj.year
                };
                let nextMonthObj = {
                    month:currentMonthObj.month === 11 ? 0 : currentMonthObj.month + 1,
                    year : currentMonthObj.month === 11 ? currentMonthObj.year + 1  : currentMonthObj.year
                };
                let preMonthLastDate = this.getMonthLastDays(preMonthObj);
                let monthFirstDay = this.getMonthFirstDay(this.currentViewMonth);
                monthFirstDay = monthFirstDay === 0 ? 7 : monthFirstDay;
                for(let i = monthFirstDay - 2 ; i >=  0 ;i--){
                    this.preMonthDays.push(moment().year(preMonthObj.year).month(preMonthObj.month).date(preMonthLastDate - i));
                }
                let currentDaysInMonth = this.currentViewMonth.daysInMonth();
                for(let i = 1 ; i <= currentDaysInMonth ; i++){
                    this.currentMonthDays.push(moment().year(currentMonthObj.year).month(currentMonthObj.month).date(i));
                }
                let currentViewMonthLastDay = this.getMonthlastDay(this.currentViewMonth);
                let remainDays = (7 - currentViewMonthLastDay);
                remainDays = remainDays === 7 ? 0 : remainDays;
                for(let i = 1 ; i <= remainDays ; i++){
                    this.nextMonthDays.push(moment().year(nextMonthObj.year).month(nextMonthObj.month).date(i));
                }
            },
            changeTable(item){
                this.currentType = item;
                this.monthHierarchyShow = false;
                this.dayHierarchyShow = false;
                switch (this.currentType){
                    case "realTime" :{
                        let current = moment();
                        this.typeListValue = current.format("YYYY年MM月DD日 hh:mm:ss");
                        this.inputValue = this.typeListValue;
                        this.$emit("input", {
                            type: item,
                            data: current
                        });
                        break;
                    }
                    case "current1":{
                        let current = moment();
                        let startDate = _.cloneDeep(current).subtract(1 , "days").startOf("day");
                        let endDate = _.cloneDeep(current).subtract(1 , "days").endOf("day");
                        this.typeListValue = startDate.format("YYYY年MM月DD日");
                        this.inputValue = this.typeListValue;
                        this.$emit("input", {
                            type: item,
                            data:{
                                startDate,
                                endDate
                            }
                        });
                        break;
                    }
                    case "current7":{
                        let current = moment();
                        let startDate = _.cloneDeep(current).subtract(7 , "days").startOf("day");
                        let endDate = _.cloneDeep(current).subtract(1 , "days").endOf("day");
                        this.typeListValue = startDate.format("YYYY年MM月DD日") + " - " + endDate.format("YYYY年MM月DD日");
                        this.inputValue = this.typeListValue;
                        this.$emit("input", {
                            type: item,
                            data:{
                                startDate,
                                endDate
                            }
                        });
                        break;
                    }
                    case "current30":{
                        let current = moment();
                        let startDate = _.cloneDeep(current).subtract(30 , "days").startOf("day");
                        let endDate = _.cloneDeep(current).subtract(1 , "days").endOf("day");
                        this.typeListValue = startDate.format("YYYY年MM月DD日") + " - " + endDate.format("YYYY年MM月DD日");
                        this.inputValue = this.typeListValue;
                        this.$emit("input", {
                            type: item,
                            data:{
                                startDate,
                                endDate
                            }
                        });
                        break;
                    }

                    case "day":
                        this.dayHierarchyShow = true;
                        this.init();
                        break;
                    case "week":
                        this.dayHierarchyShow = true;
                        this.init();
                        break;
                    case "month":

                        this.monthHierarchyShow = true;
                        this.init();
                        break;
                }
            },
            selectDay(item){
                if(this.currentType === "day"){
                    this.dayViewCurrentDay = item;
                    this.inputValue = item.format("YYYY-MM-DD");
                    this.$emit("input", {
                        type:"day",
                        data:this.dayViewCurrentDay
                    })
                }else{
                    let selectDateDay = item.day();
                    selectDateDay = selectDateDay === 0 ? 7 : selectDateDay;
                    this.weekFirstDateOfMonth = moment(_.cloneDeep(item).subtract(selectDateDay - 1, "days")).startOf("day");
                    this.weekLastDateOfMonth = moment(_.cloneDeep(item).add(7 - selectDateDay, "days")).endOf("day");
                    this.inputValue = this.weekFirstDateOfMonth.format("YYYY-MM-DD") + "至" + this.weekLastDateOfMonth.format("YYYY-MM-DD");
                    this.$emit("input", {
                        type:"week",
                        data:{
                            first : this.weekFirstDateOfMonth,
                            last : this.weekLastDateOfMonth
                        }
                    })
                }
            },
            selectMonth(item){
                this.monthViewCurrentMonth = moment(this.currentViewYear + "-" + item , "YYYY-M");
                this.$emit("input", this.monthViewCurrentMonth);
                this.inputValue = this.monthViewCurrentMonth.format("YYYY-MM");
            },
            isSameDate(d1 , d2){
                return d1.year() === d2.year() && d1.month() === d2.month() && d1.date() === d2.date()
            },
            dayActive(item){
                if(this.dayViewCurrentDay === item && this.currentType === 'day'){
                    return true;
                }else if(this.currentType === 'week'){
                    if(this.weekFirstDateOfMonth){
                        return this.isSameDate(item ,this.weekFirstDateOfMonth)  || this.isSameDate(item ,this.weekLastDateOfMonth); // true
                    }else{
                        return false;
                    }

                }else{
                    return false;
                }
            },
            dayInWeekActive(item){
                if(this.currentType === 'week'){
                    if(this.weekFirstDateOfMonth){
                        return item.isBetween(this.weekFirstDateOfMonth, this.weekLastDateOfMonth); // true
                    }else{
                        return false;
                    }

                }else{
                    return false;
                }
            },
            // select * from usersMap
//            where id in (select other_userId from friendmap where userId = ?)
            gotoPreYear(){
                this.monthViewCurrentMonth = null;
                this.currentViewYear--;
            },
            gotoNextYear(){
                this.monthViewCurrentMonth = null;
                this.currentViewYear++;
            },
            gotoPreMonth(){
                this.currentViewMonth.set( "month" , this.currentViewMonth.month() - 1);
                this.dayViewCurrentDay = null,
                this.preMonthDays = [],
                this.currentMonthDays = [],
                this.nextMonthDays = [],
                this.inputValue = null;
                this.buildCalendar();
            },
            gotoNextMonth(){
                this.currentViewMonth.set( "month" , this.currentViewMonth.month() + 1);
                this.dayViewCurrentDay = null,
                this.preMonthDays = [],
                this.currentMonthDays = [],
                this.nextMonthDays = [],
                this.inputValue = null;
                this.buildCalendar();
            },
            ViewTitle(){
                if(this.currentViewMonth){
                    return this.currentViewMonth.format("YYYY年MM月")
                }else{
                    return "";
                }
            }
        }
    }
</script>

<style scoped lang="less">
    @import "./time";
</style>
