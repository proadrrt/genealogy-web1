<template>
    <div>
        <Row :gutter="32" class="videolist">
            <i-col :span="6" v-for="v in list" :key="v.id">
                <div class="item">
                    <div class="img" :style="v.fanNewsUploadFile.length ? api.imgBG(v.fanNewsUploadFile[0].filePath) : ''">
                        <img src="http://iph.href.lu/80x45">
                        <div class="hover" @click="toShow(v.fanUploadVedioList[0].filePath)">
                            <Icon type="ios-videocam"/>
                        </div>
                    </div>
                    <div class="obj">
                        <div class="tit">{{v.title}}</div>
                        <div class="tag">
                            <span class="date">{{dayjs(v.updateTime).format("YYYY-MM-DD")}}</span>
                            <!-- <span>0 次播放</span> -->
                        </div>
                    </div>
                </div>
            </i-col>
        </Row>
        <Page :total="total" @on-change="chgPage" :page-size="8" v-if="total"/>
        <div class="modal" v-if="visible">
            <div class="bg" @click="visible = false">
                <div class="close">
                    <iconfont name="close"/>
                </div>
            </div>
            <div class="in">
                <video width="100%" controls controlslist="nofullscreen nodownload noremote footbar" autoplay v-if="curr">
                    <source :src="api.imgurl(curr)" type="video/mp4">
                </video>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    name: "VideoList",
    props: ["url"],
    data() {
        return {
            list: [],
            page: 1,
            total: 0,
            curr: "",
            visible: false
        };
    },
    watch: {
        url: function(curVal, oldVal) {
            if (curVal != oldVal) {
                this.getList();
            }
        }
    },
    mounted: function() {
        this.getList();
    },
    methods: {
        getList() {
            this.api
                .get(this.url, {
                    pageNo: this.page
                })
                .then(res => {
                    if (res.code == 200) {
                        this.list = res.data.records;
                        this.total = res.data.total;
                    }
                });
        },
        chgPage(e) {
            this.page = e;
            this.getList();
        },
        toShow(e) {
            this.curr = e;
            this.visible = true;
        }
    }
};
</script>
<style lang="scss" scoped>
@import "@/assets/css/var.scss";
.videolist {
    overflow: hidden;
    padding: 16px 0;

    .item {
        margin: 8px 0;
        position: relative;
        .img {
            position: relative;
            width: 100%;
            background: whitesmoke no-repeat center / cover;
            img {
                width: 100%;
                visibility: hidden;
                display: block;
            }
            .hover {
                position: absolute;
                top: 0;
                right: 0;
                height: 100%;
                width: 100%;
                display: flex;
                justify-content: center;
                flex-direction: column;
                text-align: center;
                background: rgba(0, 0, 0, 0.8);
                cursor: pointer;
                color: #fff;
                font-size: 48px;
                opacity: 0;
            }
            &:hover {
                .hover {
                    opacity: 1;
                }
            }
        }

        .obj {
            .tit {
                font-size: 14px;
                height: 32px;
                line-height: 32px;
                white-space: nowrap;
                overflow: hidden;
                text-overflow: ellipsis;
            }

            .tag {
                white-space: nowrap;
                font-size: 12px;
                color: #999;
                .date {
                    float: right;
                }
            }
        }
    }
}
.modal {
    position: fixed;
    top: 0;
    right: 0;
    z-index: 999;
    height: 100%;
    width: 100%;
    .bg {
        position: relative;
        height: 100%;
        width: 100%;
        background: rgba(black, 0.8);
        .close {
            position: absolute;
            z-index: 1;
            cursor: pointer;
            right: 5%;
            top: 5%;
            i {
                font-size: 40px;
                color: #fff;
                opacity: 0.5;
                &:hover {
                    opacity: 1;
                }
            }
        }
    }
    .in {
        position: absolute;
        z-index: 1;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        video {
            min-width: 600px;
            max-width: 90%;
            max-height: 600px;
            min-height: 300px;
        }
    }
}
</style>
