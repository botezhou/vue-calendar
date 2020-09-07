<template>
    <div class="calendar-wrap">
        <calendar @checkedDay="checkedDay" @monthChange="monthChange" :month-record-data="monthRecordData"/>
        <div class="order-record-wrap">
            <p class="ts-msg" v-if="dayData.length==0">当前日期暂无活动！！！</p>
            <div class="record-list" v-else>
                <ul>
                    <li class="record-item" v-for="(item,i) in dayData" :key="i">
                        <van-icon class="symbol" name="award" />
                        <div class="thing-box">
                            <h6 class="thing-title">{{item.title}} <span class="ml10">{{ item.startTime | timeFormat}}</span>-<span class="ml10">{{ item.endTime | timeFormat}}</span></h6>
                            <p class="thing-desc">{{item.content}}</p>
                        </div>
                        <div class="handle-box">
                            <van-icon class="icon" @click="editClick(item)" name="edit" />
                            <van-icon class="icon" @click="delClick(item)" name="delete" />
                        </div>
                    </li>
                </ul>
            </div>
            <van-icon class="add-icon" @click="addCalenDarClick" name="add" />
        </div>
        <van-dialog v-model="dialogShow" @confirm="onSubmit" title="日程管理" show-cancel-button>
            <div>
                <van-field
                    v-model="editData.title"
                    name="日程标题"
                    label="日程标题"
                    placeholder="日程标题"
                />
                <van-field
                    v-model="editData.content"
                    type="textarea"
                    rows="1"
                    autosize
                    name="日程内容"
                    label="日程内容"
                    placeholder="日程内容"
                />
                <van-field
                    readonly
                    clickable
                    name="datetimePicker"
                    :value="editData.startTime"
                    label="开始时间"
                    placeholder="点击选择时间"
                    @click="timePicker(editData.startTime,'startTime')"
                />
                <van-field
                    readonly
                    clickable
                    name="datetimePicker"
                    :value="editData.endTime"
                    label="结束时间"
                    placeholder="点击选择时间"
                    @click="timePicker(editData.endTime,'endTime')"
                />
            </div>
        </van-dialog>
        <van-popup v-model="showPicker" position="bottom">
            <van-datetime-picker
                v-model="currentTime"
                type="time"
                @confirm="onTimeConfirm"
                @cancel="showPicker = false"
            />
        </van-popup>
    </div>
</template>

<script>
import calendar from '@/components/calendar'
// import { getMonthData, getDateData, addCalenDar, deleteCalenDar } from '@/api/api'

function timeFormater(originVal){
    const dt = new Date(originVal)
    const y = dt.getFullYear()
    // padStart 填充到多少位，填充内容为'0'
    const m = (dt.getMonth() + 1 + '').padStart(2, '0');
    const d = (dt.getDate() + '').padStart(2, '0');

    const hh = (dt.getHours() + '').padStart(2, '0');
    const mm = (dt.getMinutes() + '').padStart(2, '0');
    const ss = (dt.getSeconds() + '').padStart(2, '0');
    return `${hh}:${mm}`
}
export default {
    components: {
        calendar
    },
    data () {
        return {
            dialogShow: false,
            showPicker: false,
            timeTypePicker: '',
            currentTime: '12:00',
            checkedDate: '',
            dayParmas: {},
            monthRecordData: [],
            dayData: [],
            editData: {
                title: '',
                content: '',
                startTime: '',
                endTime: ''
            }
        }
    },
    created () {
        this._initCurrentDate()
    },
    methods: {
        _initCurrentDate(){
            const date = new Date();
            const year = date.getFullYear();
            const month = (date.getMonth() + 1 + '').padStart(2, '0');
            const day = (date.getDate() + '').padStart(2, '0');
            const parmas = {
                YEAR: year,
                MONTH: month
            }
            this.checkedDate = `${year}-${month}-${day}`;
            this.dayParmas = {
                YEAR: year,
                MONTH: Number(month),
                DAY: Number(day)
            }
            // this._getMonthData(parmas)
            // this._getDayData(this.dayParmas)
        },
        // _getMonthData(parmas) {
        //     getMonthData(parmas)
        //         .then(res => {
        //             if(res.rcode=='0000'){
        //                 this.monthRecordData = res.data
        //             } else {
        //                 this.$toast.fail('数据加载失败');
        //             }
        //         })
        // },
        // _getDayData(parmas) {
        //     getDateData(parmas)
        //         .then(res => {
        //             console.log(res)
        //             if(res.rcode=='0000'){
        //                 this.dayData = res.data
        //             } else {
        //                 this.$toast.fail('数据加载失败');
        //             }
        //         })
        // },

        // 点击日历获取当天数据
        checkedDay(date){
            this.checkedDate = date;
            const currentDate = date.split('-');
            const year = currentDate[0]
            const month = Number(currentDate[1])
            const day = Number(currentDate[2])
            this.dayParmas = {
                YEAR: year,
                MONTH: month,
                DAY: day
            }
            // this._getDayData(this.dayParmas)
        },
        // 切换月份获取月份数据
        monthChange(date){
            const Date = date.split('-');
            const year = Date[0];
            const month = Number(Date[1])
            const parmas = {
                YEAR: year,
                MONTH: month
            }
            // this._getMonthData(parmas)
        },
        // 打开时间选择器
        timePicker(time,type){
            this.currentTime = time;
            this.showPicker = true;
            this.timeTypePicker = type;
        },
        // 修改数据
        editClick(item){
            this.editData = JSON.parse((JSON.stringify(item)));
            this.editData.startTime = timeFormater(item.startTime)
            this.editData.endTime = timeFormater(item.endTime)
            this.dialogShow = true
        },
        // 提交数据修改
        onSubmit(){
            const startTime = new Date(this.checkedDate+' '+this.editData.startTime);
            const endTime = new Date(this.checkedDate+' '+this.editData.endTime);
            const params = {
                id: this.editData.id?this.editData.id: '',
                title: this.editData.title,
                content: this.editData.content,
                startTime: startTime,
                endTime: endTime
            }
            addCalenDar(params)
                .then(res => {
                    if(res.rcode=='0000'){
                        this.$toast.success('操作成功');
                        // this._getDayData(this.dayParmas)
                    } else {
                        this.dialogShow = true
                        this.$toast.fail(res.desc);
                    }
                })
        },
        // 添加日程
        addCalenDarClick(){
            this.dialogShow = true;
            this.editData = {
                id: '',
                title: '',
                content: '',
                startTime: '12: 00',
                endTime: '18: 00'
            }
        },
        // 删除日程
        delClick(item){
            this.$dialog.confirm({
              title: '',
              message: '确定删除此日程吗？',
            })
            .then(() => {
                const params = {id: item.id};
                // deleteCalenDar(params)
                //     .then(res => {
                //         if(res.rcode=='0000'){
                //             this.$toast.success('操作成功');
                //             this._getDayData(this.dayParmas)
                //         } else {
                //             this.dialogShow = true
                //             this.$toast.fail(res.desc);
                //         }
                //     })
            })
            .catch(() => {

            });

        },
        onTimeConfirm(){
            this.showPicker = false;
            this.editData[this.timeTypePicker] = this.currentTime
        }
    }
}
</script>

<style lang="less">
    .calendar-wrap{
        .order-record-wrap{
            background: #fff;
            padding: 10px 10px 40px;
            margin-top: 10px;
            position: relative;
            .add-icon{
                font-size: 30px;
                position: absolute;
                right: 10px;
                bottom: 10px;
                color: #f08d49;
            }
            .ts-msg{
                color: #666;
                font-size: 14px;
                padding: 5px;
            }
            .record-item{
                position: relative;
                padding: 5px 0 10px;
                margin-bottom: 10px;
                border-bottom: 1px dashed #f1f1f1;
                .symbol{
                   position: absolute;
                   left: 0;
                   top: 3px;
                   font-size: 18px;
                   color: #f08d49;
                }
                .thing-box{
                    padding-left: 24px;
                    .thing-title{
                        font-size: 14px;
                        color: #333;
                        margin-bottom: 10px;
                    }
                    .thing-desc{
                        font-size: 12px;
                        color: #666;
                    }
                }
                .handle-box{
                    position: absolute;
                    top: 5px;
                    right: 0;
                    .icon{
                        font-size: 16px;
                        margin-right: 10px;
                        cursor: pointer;
                    }
                }
            }
        }
        .van-dialog__header{
            background: #f08d49;
            color: #fff;
            padding: 10px 0;
        }

    }

</style>
