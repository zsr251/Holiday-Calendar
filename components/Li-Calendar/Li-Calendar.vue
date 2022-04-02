<template>
	<view class="bg-white">
		<view class='calendar-Time-header under_line'>
			<text class='calendar-lastMonth' @tap="subMonth"
				v-bind:class="{'calendar-btn-disabled' : lastDisabled }"><text class="cuIcon-back"></text></text>
			<picker mode="date" @change="dateChange">
				<text class='calendar-TimeH'>{{title_time}}</text>
			</picker>
			<text class='calendar-nextMonth' @tap="addMonth"
				v-bind:class="{'calendar-btn-disabled' : nextDisabled }"><text class="cuIcon-right"></text></text>
			<text class='calendar-backToToday text-lg' @tap="backToToday"> <text class="round cuIcon-focus"></text> </text>
		</view>
		<view class="calendar-content" >
			<view class="calendar-row calendar-header">
				<view class="calendar-col aligncanter">日</view>
				<view class="calendar-col aligncanter">一</view>
				<view class="calendar-col aligncanter">二</view>
				<view class="calendar-col aligncanter">三</view>
				<view class="calendar-col aligncanter">四</view>
				<view class="calendar-col aligncanter">五</view>
				<view class="calendar-col aligncanter">六</view>
			</view>
			<view class="calendar-row"
				v-bind:style="{'transform':transformObj, 'transition-duration':transformTimeObj}"
				@touchstart="touchstart" @touchmove="touchmove" @touchend="touchend">
				<view class="calendar-col aligncanter calendar-col-lastMonth" v-for='item in beforeDateList'
					v-bind:key='item.key' @tap="subMonth">
					<view class='dayValue'>
						<text class='calendar-date'>{{item.dateIndex}}</text>
						<text class='calendar-point' v-bind:style="{'color':item.pointTextColor?item.pointTextColor:maskColor}">{{item.pointText}}</text>
						<text class='calendar-text'>{{item.markText}}</text>
					</view>
				</view>

				<view class="calendar-col aligncanter currentDays" v-bind:class="{'calendar-today' : item.isToday ,'calendar-active' : !item.isToday && item.dateIndex == currentSelectTime }" 
				v-for='item in dateList' v-bind:key='item.key'
					@tap="selectedDateFun(item.dateIndex)">
					<view class='dayValue'>
						<text class='calendar-date' :class="{'calendar-day-6-7':(item.isDaySunOrSat && item.pointText != '班')}">{{item.dateIndex}}</text>
						<text class='calendar-point round' v-if="item.pointText"><text class="text-xs text-bold text-shadow" :class="{'text-blue':item.pointText == '班','text-red':item.pointText == '休'}">{{item.pointText}}</text></text>
						<text class='calendar-text' v-bind:style="{'color':item.markTextColor?item.markTextColor:maskColor}">{{item.markText}}</text>
					</view>
				</view>

				<view class="calendar-col aligncanter calendar-col-nextMonth" v-for='item in afterDateList'
					v-bind:key='item.key' @tap="addMonth">
					<view class='dayValue'>
						<text class='calendar-date'>{{item.dateIndex}}</text>
						<text class='calendar-point' v-bind:style="{'color':item.pointTextColor?item.pointTextColor:maskColor}">{{item.pointText}}</text>
						<text class='calendar-text'>{{item.markText}}</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	const {
		Solar,
		SolarMonth,
		Lunar,
		HolidayUtil
	} = require('@/common/lunar.js');

	export default {
		name: 'Li-Calendar',
		props: {
			/**
			 * @description 当前时间 yyyy-MM-dd
			 */
			currentTime: {
				type: String,
				default: function() {
					let timeObj = new Date();
					return timeObj.getFullYear() + "-" + (timeObj.getMonth() + 1) + "-" + timeObj.getDate();
				}
			},
			/**
			 * @description 全局标记点颜色，包括点和文字
			 */
			maskColor: {
				type: String,
				default: '#e54d42'
			},
			/**
			 * @description 是否显示农历，mask优先级高
			 */
			showLunar: {
				type: Boolean,
				default: true
			},
			/**
			 * @description 是否可以滚动
			 */
			canDrag: {
				type: Boolean,
				default: true
			},
		},
		created() {
			this.render();
		},
		data() {
			return {
				nextDisabled: false,
				lastDisabled: false,
				currentSelectTime: undefined, //点击时保存的dateIndex
				title_time: '', //顶头文本
				year: undefined, //当前年
				month: undefined, //当前月
				beforeDateList: [], //上个月的日期列表
				dateList: [], //本月的日期列表
				afterDateList: [], //下个月的日期列表
				transform_x: 0,
				transform_time: 0,
			};
		},
		methods: {
			render(setTimeStr) { //初始化
				let that = this;
				let currentTimeStr = setTimeStr ? setTimeStr : that.currentTime;
				// console.log(currentTimeStr)
				let timeObj = this.toDateByStr(currentTimeStr); //当前选定的时间
				let m = SolarMonth.fromDate(timeObj);
				let t = Solar.fromDate(new Date());
				// 创建前面的日期
				that.beforeDateList = [];
				that.dateList = [];
				that.afterDateList = [];
				that.year = m.getYear(); that.month = m.getMonth();
				//计算头部显示的年月
				that.title_time = m.getYear() + "年" + m.getMonth() + "月";
				// console.log(m.toFullString())
				let weeks = m.getWeeks(0);
				// console.log(weeks)
				for (let i = 0, j = weeks.length; i < j; i++) {
					let days = weeks[i].getDays();
					for (let k = 0, l = days.length; k < l; k++) {
						let tempObj = {
							isToday: false
						};
						// console.log(days[k])
						let lu = days[k].getLunar();
						// 周末
						if (k == 0 || k == 6){
							tempObj.isDaySunOrSat = true;
						}
						// 节假日
						let ho = HolidayUtil.getHoliday(days[k].toYmd());
						if(ho){
							if(ho.isWork()){
								// 调休
								tempObj.pointText = '班';
								tempObj.pointTextColor = '#0081ff';
							}else{
								// 放假
								tempObj.pointText = '休';
								tempObj.pointTextColor = '#e54d42';
							}
							if(ho.getTarget() == days[k].toYmd()){
								tempObj.markText = ho.getName();
								tempObj.markTextColor = '#e54d42';
							}
						}
						// 节气
						if(!tempObj.markText){
							let jie = lu.getJieQi();
							if(jie){
								tempObj.markText = jie;
								tempObj.markTextColor = '#e54d42';
								// console.log(jie);
							}
						}
						// 农历
						if(!tempObj.markText){
							// 农历
							let tempLunarDay = lu.getDayInChinese() == '初一' ? lu.getMonthInChinese() + '月' : lu.getDayInChinese();
							tempObj.markText = tempLunarDay;
							tempObj.markTextColor = '#8799a3';
						}
						// 当日
						if (days[k].getYear() == t.getYear() && days[k].getMonth() == t.getMonth() && days[k].getDay() == t.getDay()) {
							tempObj.isToday = true;
						}
						if (m.getYear() > days[k].getYear() || (m.getYear() == days[k].getYear() && m.getMonth() > days[k].getMonth())) {
							tempObj.key = 'before_' + days[k].getDay();
							tempObj.dateIndex = days[k].getDay();
							that.beforeDateList.push(tempObj);
							// console.log('当月前' + tempObj.key)
						} else if (m.getYear() < days[k].getYear() || (m.getYear() == days[k].getYear() && m.getMonth() < days[k].getMonth())) {
							tempObj.key = 'after_' + days[k].getDay();
							tempObj.dateIndex = days[k].getDay();
							that.afterDateList.push(tempObj);
							// console.log('当月后' + tempObj.key)
						} else {
							tempObj.key = 'date_' + days[k].getDay();
							tempObj.dateIndex = days[k].getDay();
							that.dateList.push(tempObj);
							// console.log('当月' + tempObj.key)
						}

					}
				}
			},
			dateChange(e){
				let selectDate = undefined;
				if(e.type == 'change'){
					selectDate = new Date(e.detail.value); 
				}
				if(!selectDate){
					return;
				}
				//如果已经是当月
				if (selectDate.getFullYear() == this.year && (selectDate.getMonth() + 1) == this.month && selectDate.getDate() == t.currentSelectTime) {
					console.log('无变化 不需要加载')
					return;
				}
				let value = selectDate.getFullYear() + "/" + (selectDate.getMonth() + 1) + "/" + selectDate.getDate();
				this.$emit('monthChange', {
					date: value
				});
				this.render(value);
				this.selectedDateFun(selectDate.getDate());
			},
			toDateByStr(timeStr) { //字符串转换时间，转换失败或者不传字符串则返回当前
				let timeObj;
				if (timeStr) {
					timeObj = new Date(timeStr.replace(/-/g, "/"));
				}
				if (!timeStr || timeObj == "Invalid Date")
					timeObj = new Date();
				return timeObj;
			},
			selectedDateFun(index) { //点击日期
				let that = this;
				let selectObj = that.dateList[index - 1];
				selectObj.year = that.year;
				selectObj.month = that.month;
				selectObj.day = index;
				selectObj.time = that.year + "/" + that.month + "/" + index;
				that.$emit('dayChange', selectObj);
				that.currentSelectTime = index;

			},
			addMonth() { //加一个月
				let month = SolarMonth.fromYm(this.year,this.month);
				month = month.next(1);
				let value = month.getYear() + "/" + month.getMonth() + "/1" ;
				
				this.$emit('monthChange', {
					date: value
				});
				this.render(value);
				this.selectedDateFun(1);
			},
			subMonth() { //减一个月
				let month = SolarMonth.fromYm(this.year,this.month);
				month = month.next(-1);
				let value = month.getYear() + "/" + month.getMonth() + "/1" ;
				
				this.$emit('monthChange', {
					date: value
				});
				this.render(value);
				this.selectedDateFun(1);
			},
			backToToday() { //回到今天
				let currDate = new Date();
				let _year = currDate.getFullYear();
				let _month = currDate.getMonth() + 1;
				let _day = currDate.getDate();

				//如果已经是当月
				if (_year == this.year && _month == this.month && _day == this.currentSelectTime) {
					console.log('无变化 不需要加载')
					return;
				}

				let value = _year + "/" + _month + "/" + _day;
				
				this.$emit('monthChange', {
					date: value
				});
				this.render(value);
				this.selectedDateFun(_day);
			},
		
			calendarTransform(x, time) { //日历滑动动画
				this.transform_x = x;
				this.transform_time = time;
			},
			touchstart(event) {
				if (!this.canDrag)
					return;
				this.startPageX = event.touches[0].pageX;
				this.startPageY = event.touches[0].pageY;
			},
			touchmove(event) {
				if (!this.canDrag)
					return;
				let touchmovePageX = event.touches[0].pageX;
				let result = touchmovePageX - this.startPageX;
				this.calendarTransform(result, 0);
			},
			touchend(event) {
				if (!this.canDrag)
					return;
				let that = this;
				let endPageX = event.changedTouches[0].pageX;
				let endPageY = event.changedTouches[0].pageY;
				let x = Math.abs(that.startPageX - endPageX); //横坐标之差
				let y = Math.abs(that.startPageY - endPageY); //纵坐标之差

				let screenX = 0;
				try {
					const res = uni.getSystemInfoSync();
					screenX = res.windowWidth;
				} catch (e) {
					console.error(e)
					return;
				}
				if (that.startPageX > endPageX) { //左滑
					let a = Math.atan(y / x);
					let Rate = x / screenX;
					if (a < Math.PI / 6 && Rate > 0.3) {
						screenX = -screenX;
						that.calendarTransform(screenX, 300);
						setTimeout(function() {
							that.addMonth();
							that.calendarTransform(0, 0);
						}, 300);
					} else {
						that.calendarTransform(0, 300);
					}
				} else {
					let a = Math.atan(y / x);
					let Rate = x / screenX;
					if (a < Math.PI / 6 && Rate > 0.3) {
						that.calendarTransform(screenX, 300);
						setTimeout(function() {
							that.subMonth();
							that.calendarTransform(0, 0);
						}, 300);
					} else {
						that.calendarTransform(0, 300);
					}
				}
			}
		},
		computed: {
			transformObj: function() {
				return 'translate3d(' + this.transform_x + 'px, 0px, 0px) translateZ(0px)';
			},
			transformTimeObj: function() {
				return this.transform_time + 'ms';
			}
		},
	}
</script>

<style>
	@import url("./Li-Calendar.css");
</style>
