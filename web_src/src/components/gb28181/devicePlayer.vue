<template>
<div id="devicePlayer">
    <el-dialog title="视频播放" top="0" :close-on-click-modal="false" :visible.sync="showVideoDialog" :destroy-on-close="true" @close="close()">
        <LivePlayer v-if="showVideoDialog" ref="videoPlayer" :videoUrl="videoUrl" :error="videoError" :hasaudio="hasaudio" fluent autoplay live></LivePlayer>
        <div id="shared" style="text-align: right; margin-top: 1rem;">
            <el-tabs v-model="tabActiveName">
                <el-tab-pane label="实时视频" name="media">
                    <div style="margin-bottom: 0.5rem;">
                        <!--		<el-button type="primary" size="small" @click="playRecord(true, '')">播放</el-button>-->
                        <!--		 <el-button type="primary" size="small" @click="startRecord()">录制</el-button>-->
                        <!--		 <el-button type="primary" size="small" @click="stopRecord()">停止录制</el-button>-->
                    </div>
                    <div style="display: flex; margin-bottom: 0.5rem; height: 2.5rem;">
                        <span style="width: 5rem; line-height: 2.5rem; text-align: right;">播放地址：</span>
                        <el-input v-model="getPlayerShared.sharedUrl" :disabled="true" v-on:click.native="copySharedInfo(getPlayerShared.sharedUrl)"></el-input>
                    </div>
                    <div style="display: flex; margin-bottom: 0.5rem; height: 2.5rem;">
                        <span style="width: 5rem; line-height: 2.5rem; text-align: right;">iframe：</span>
                        <el-input v-model="getPlayerShared.sharedIframe" :disabled="true" v-on:click.native="copySharedInfo(getPlayerShared.sharedIframe)"></el-input>
                    </div>
                    <div style="display: flex; margin-bottom: 0.5rem; height: 2.5rem;">
                        <span style="width: 5rem; line-height: 2.5rem; text-align: right;">资源地址：</span>
                        <el-input v-model="getPlayerShared.sharedRtmp" :disabled="true" v-on:click.native="copySharedInfo(getPlayerShared.sharedRtmp)"></el-input>
                    </div>
                </el-tab-pane>
                <!--{"code":0,"data":{"paths":["22-29-30.mp4"],"rootPath":"/home/kkkkk/Documents/ZLMediaKit/release/linux/Debug/www/record/hls/kkkkk/2020-05-11/"}}-->
                <el-tab-pane label="录像查询" name="record">
                    <el-date-picker size="mini" v-model="videoHistory.date" type="date" value-format="yyyy-MM-dd" placeholder="日期" @change="queryRecords()"></el-date-picker>
                    <!--            <el-slider style="margin: 0 1rem 1rem 1rem;"-->
                    <!--                       v-model="timeVal"-->
                    <!--                       :min="timeMin"-->
                    <!--                       :max="timeMax"-->
                    <!--                       :step="5"-->
                    <!--                       :marks="getTimeMakrs()"-->
                    <!--                       :format-tooltip="formatTooltip">-->
                    <!--            </el-slider>-->
                    <!--            <range-slider :min="timeMin"-->
                    <!--                          :max="timeMax"-->
                    <!--                          :step="5"></range-slider>-->

                    <!--		<el-date-picker v-model="videoHistory.endTime" type="datetime" value-format="yyyy-MM-dd HH:mm:ss" placeholder="结束时间"-->
                    <!--		 @change="recordList()"></el-date-picker>-->
                    <el-table :data="videoHistory.searchHistoryResult" height="150" v-load="recordsLoading">
                        <el-table-column label="名称" prop="name"></el-table-column>
                        <el-table-column label="文件" prop="filePath"></el-table-column>
                        <el-table-column label="开始时间" prop="startTime" :formatter="timeFormatter"></el-table-column>
                        <el-table-column label="结束时间" prop="endTime" :formatter="timeFormatter"></el-table-column>

                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button icon="el-icon-video-play" size="mini" @click="playRecord(scope.row)">播放</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <!--遥控界面-->
                <el-tab-pane label="云台控制" name="control">
                    <div style="display: flex; justify-content: center;">
                        <div class="control-wrapper">
                            <div class="control-btn control-top" @mousedown="ptzCamera(0, 1, 0)" @mouseup="ptzCamera(0, 0, 0)">
                                <i class="el-icon-caret-top"></i>
                                <div class="control-inner-btn control-inner"></div>
                            </div>
                            <div class="control-btn control-left" @mousedown="ptzCamera(1, 0, 0)" @mouseup="ptzCamera(0, 0, 0)">
                                <i class="el-icon-caret-left"></i>
                                <div class="control-inner-btn control-inner"></div>
                            </div>
                            <div class="control-btn control-bottom" @mousedown="ptzCamera(0, 2, 0)" @mouseup="ptzCamera(0, 0, 0)">
                                <i class="el-icon-caret-bottom"></i>
                                <div class="control-inner-btn control-inner"></div>
                            </div>
                            <div class="control-btn control-right" @mousedown="ptzCamera(2, 0, 0)" @mouseup="ptzCamera(0, 0, 0)">
                                <i class="el-icon-caret-right"></i>
                                <div class="control-inner-btn control-inner"></div>
                            </div>
                            <div class="control-round">
                                <div class="control-round-inner"><i class="fa fa-pause-circle"></i></div>
                            </div>
                            <div style="position: absolute; left: 7.25rem; top: 1.25rem" @mousedown="ptzCamera(0, 0, 2)" @mouseup="ptzCamera(0, 0, 0)"><i class="el-icon-zoom-in" style="font-size: 1.875rem;"></i></div>
                            <div style="position: absolute; left: 7.25rem; top: 3.25rem; font-size: 1.875rem;" @mousedown="ptzCamera(0, 0, 1)" @mouseup="ptzCamera(0, 0, 0)"><i class="el-icon-zoom-out"></i></div>
                        </div>
                    </div>

                </el-tab-pane>
            </el-tabs>
        </div>
    </el-dialog>
</div>
</template>

<script>
import LivePlayer from '@liveqing/liveplayer'
export default {
    name: 'devicePlayer',
    props: {},
    components: {
        LivePlayer
    },
    computed: {
        getPlayerShared: function () {
            return {
                sharedUrl: window.location.host + '/' + this.videoUrl,
                sharedIframe: '<iframe src="' + window.location.host + '/' + this.videoUrl + '"></iframe>',
                sharedRtmp: this.videoUrl
            };
        }
    },
    created() {},
    data() {
        return {
            video: 'http://lndxyj.iqilu.com/public/upload/2019/10/14/8c001ea0c09cdc59a57829dabc8010fa.mp4',
            videoUrl: '',
            videoHistory: {
                date: '',
                searchHistoryResult: [] //媒体流历史记录搜索结果
            },
            timeMakrs: {
                // 0	: "0:00",
                // // 60	: "1:00",
                // 120	: "2:00",
                // // 180	: "3:00",
                // 240	: "4:00",
                // // 300 : "5:00",
                // 360	: "6:00",
                // // 420	: "7:00",
                // 480	: "8:00",
                // 540	: "9:00",
                600: "10:00",
                // 660	: "11:00",
                720: "12:00",
                // 780	: "13:00",
                840: "14:00",
                // 900 : "15:00",
                960: "16:00",
                // 1020	: "17:00",
                1080: "18:00",
                // 1140	: "19:00",
                // 1200  : "20:00",
                // // 1260	: "21:00",
                // 1320	: "22:00",
                // // 1380	: "23:00",
                // 1440	: "24:00"
            },
            showVideoDialog: false,
            ssrc: '',
            deviceId: '',
            channelId: '',
            tabActiveName: 'media',
            hasaudio: false,
            loadingRecords: false,
            recordsLoading: false,
            timeVal: 0,
            timeMin: 0,
            timeMax: 1440,

        };
    },
    methods: {
        openDialog: function (tab, deviceId, channelId, param) {
            this.tabActiveName = tab;
            this.channelId = channelId;
            this.deviceId = deviceId;
            this.ssrc = "";
            this.videoUrl = ""
            if (!!this.$refs.videoPlayer) {
                this.$refs.videoPlayer.pause();
            }

            switch (tab) {
                case "media":
                    this.play(param.streamInfo, param.hasAudio)
                    break;
                case "record":
                    this.showVideoDialog = true;

                    this.videoHistory.date = param.date;
                    this.queryRecords()
                    break;
                case "control":
                    break;
            }
        },
        timeAxisSelTime: function (val) {
            console.log(val)
        },
        getTimeMakrs() {
            return this.timeMakrs;
        },
        play: function (streamInfo, hasAudio) {
            this.hasaudio = hasAudio;
            // 根据媒体流信息二次判断
            if (!!streamInfo.tracks && streamInfo.tracks.length > 0) {
                var realHasAudio = false;
                for (let i = 0; i < streamInfo.tracks.length; i++) {
                    if (streamInfo.tracks[i].codec_type == 1 && streamInfo.tracks[i].codec_id_name == "CodecAAC") { // 判断为AAC音频
                        realHasAudio = true;
                    }
                }
                this.hasaudio = realHasAudio && this.hasaudio;
            }
            this.ssrc = streamInfo.ssrc;
            // this.$refs.videoPlayer.hasaudio = hasAudio;
            // this.videoUrl = streamInfo.flv + "?" + new Date().getTime();
            this.videoUrl = streamInfo.ws_flv;
            this.showVideoDialog = true;
            console.log(this.ssrc);
        },
        close: function () {
            console.log('关闭视频');
            this.$refs.videoPlayer.pause();
            this.videoUrl = '';
            this.showVideoDialog = false;
        },
        copySharedInfo: function (data) {
            console.log('复制内容：' + data);
            let _this = this;
            this.$copyText(data).then(
                function (e) {
                    _this.$message({
                        showClose: true,
                        message: '复制成功',
                        type: 'success'
                    });
                },
                function (e) {
                    _this.$message({
                        showClose: true,
                        message: '复制失败，请手动复制',
                        type: 'error'
                    });
                }
            );
        },

        queryRecords: function () {
            if (!this.videoHistory.date) {
                return;
            }
            this.recordsLoading = true;
            this.videoHistory.searchHistoryResult = [];
            let that = this;
            var startTime = this.videoHistory.date + " 00:00:00";
            var endTime = this.videoHistory.date + " 23:59:59";
            this.$axios({
                method: 'get',
                url: '/api/record/' + this.deviceId + '/' + this.channelId + '?startTime=' + startTime + '&endTime=' + endTime
            }).then(function (res) {
                // 处理时间信息
                that.videoHistory.searchHistoryResult = res.data.recordList;
                that.recordsLoading = false;
            }).catch(function (e) {
                console.log(e.message);
                // that.videoHistory.searchHistoryResult = falsificationData.recordData;
            });

        },
        onTimeChange: function (video) {
            // this.queryRecords()
        },
        playRecord: function (row) {
            let that = this;
            if (that.ssrc != "") {
                that.stopPlayRecord(function () {
                    that.ssrc = "",
                        that.playRecord(row);
                })
            } else {
                this.$axios({
                    method: 'get',
                    url: '/api/playback/' + this.deviceId + '/' + this.channelId + '?startTime=' + row.startTime + '&endTime=' +
                        row.endTime
                }).then(function (res) {
                    var streamInfo = res.data;
                    that.ssrc = streamInfo.ssrc;
                    that.videoUrl = streamInfo.ws_flv;
                });
            }
        },
        stopPlayRecord: function (callback) {
            this.$refs.videoPlayer.pause();
            this.videoUrl = '';
            this.$axios({
                method: 'get',
                url: '/api/playback/' + this.ssrc + '/stop'
            }).then(function (res) {
                if (callback) callback()
            });
        },
        ptzCamera: function (leftRight, upDown, zoom) {
            console.log('云台控制：' + leftRight + ' : ' + upDown + " : " + zoom);
            let that = this;
            this.$axios({
                method: 'post',
                url: '/api/ptz/' + this.deviceId + '/' + this.channelId + '?leftRight=' + leftRight + '&upDown=' + upDown +
                    '&inOut=' + zoom + '&moveSpeed=50&zoomSpeed=50'
            }).then(function (res) {});
        },
        //////////////////////播放器事件处理//////////////////////////
        videoError: function (e) {
            console.log("播放器错误：" + JSON.stringify(e));
        },
        formatTooltip: function (val) {
            var h = parseInt(val / 60);
            var hStr = h < 10 ? ("0" + h) : h;
            var s = val % 60;
            var sStr = s < 10 ? ("0" + s) : s;
            return h + ":" + sStr;
        },
        timeFormatter: function (row, column, cellValue, index) {
            return cellValue.split(" ")[1];
        },
        mergeTime: function (timeArray) {
            var resultArray = [];
            for (let i = 0; i < timeArray.length; i++) {
                var startTime = new Date(timeArray[i].startTime);
                var endTime = new Date(timeArray[i].endTime);
                if (i == 0) {
                    resultArray[0] = {
                        startTime: startTime,
                        endTime: endTime
                    }
                }
                for (let j = 0; j < resultArray.length; j++) {
                    if (startTime > resultArray[j].endTime) { // 合并
                        if (startTime - resultArray[j].endTime <= 1000) {
                            resultArray[j].endTime = endTime;
                        } else {
                            resultArray[resultArray.length] = {
                                startTime: startTime,
                                endTime: endTime
                            }
                        }
                    } else if (resultArray[j].startTime > endTime) { // 合并
                        if (resultArray[j].startTime - endTime <= 1000) {
                            resultArray[j].startTime = startTime;
                        } else {
                            resultArray[resultArray.length] = {
                                startTime: startTime,
                                endTime: endTime
                            }
                        }
                    }
                }
            }
            console.log(resultArray)
            return resultArray;
        }
    }
};
</script>

<style>
.control-wrapper {
    position: relative;
    width: 6.25rem;
    height: 6.25rem;
    max-width: 6.25rem;
    max-height: 6.25rem;
    margin: 0 auto;
    border-radius: 100%;
    float: left;
}

.control-btn {
    display: flex;
    justify-content: center;
    position: absolute;
    width: 44%;
    height: 44%;
    border-radius: 5px;
    border: 1px solid #78aee4;
    box-sizing: border-box;
    transition: all 0.3s linear;
}

.control-btn i {
    font-size: 20px;
    color: #78aee4;
    display: flex;
    justify-content: center;
    align-items: center;
}

.control-round {
    position: absolute;
    top: 21%;
    left: 21%;
    width: 58%;
    height: 58%;
    background: #fff;
    border-radius: 100%;
}

.control-round-inner {
    position: absolute;
    left: 15%;
    top: 15%;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 70%;
    height: 70%;
    font-size: 40px;
    color: #78aee4;
    border: 1px solid #78aee4;
    border-radius: 100%;
    transition: all 0.3s linear;
}

.control-inner-btn {
    position: absolute;
    width: 60%;
    height: 60%;
    background: #fafafa;
}

.control-top {
    top: -8%;
    left: 27%;
    transform: rotate(-45deg);
    border-radius: 5px 100% 5px 0;
}

.control-top i {
    transform: rotate(45deg);
    border-radius: 5px 100% 5px 0;
}

.control-top .control-inner {
    left: -1px;
    bottom: 0;
    border-top: 1px solid #78aee4;
    border-right: 1px solid #78aee4;
    border-radius: 0 100% 0 0;
}

.control-top .fa {
    transform: rotate(45deg) translateY(-7px);
}

.control-left {
    top: 27%;
    left: -8%;
    transform: rotate(45deg);
    border-radius: 5px 0 5px 100%;
}

.control-left i {
    transform: rotate(-45deg);
}

.control-left .control-inner {
    right: -1px;
    top: -1px;
    border-bottom: 1px solid #78aee4;
    border-left: 1px solid #78aee4;
    border-radius: 0 0 0 100%;
}

.control-left .fa {
    transform: rotate(-45deg) translateX(-7px);
}

.control-right {
    top: 27%;
    right: -8%;
    transform: rotate(45deg);
    border-radius: 5px 100% 5px 0;
}

.control-right i {
    transform: rotate(-45deg);
}

.control-right .control-inner {
    left: -1px;
    bottom: -1px;
    border-top: 1px solid #78aee4;
    border-right: 1px solid #78aee4;
    border-radius: 0 100% 0 0;
}

.control-right .fa {
    transform: rotate(-45deg) translateX(7px);
}

.control-bottom {
    left: 27%;
    bottom: -8%;
    transform: rotate(45deg);
    border-radius: 0 5px 100% 5px;
}

.control-bottom i {
    transform: rotate(-45deg);
}

.control-bottom .control-inner {
    top: -1px;
    left: -1px;
    border-bottom: 1px solid #78aee4;
    border-right: 1px solid #78aee4;
    border-radius: 0 0 100% 0;
}

.control-bottom .fa {
    transform: rotate(-45deg) translateY(7px);
}
</style>
