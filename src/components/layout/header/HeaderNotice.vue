<template>
    <a-popover class="header-notice" trigger="click" placement="bottomRight">
        <template slot="content">
            <a-spin class="header-notice-content" :spinning="loadding">
                <a-tabs :tabBarGutter="25">
                    <a-tab-pane key="1">
                        <span slot="tab">通知<span
                                v-if="total && totalSum['notice']">({{totalSum['notice']}})</span></span>
                        <template v-if="total && totalSum['notice']">
                            <a-list>
                                <template v-for="item in list['notice']">
                                    <a-list-item>
                                        <a-list-item-meta :description="item.create_time">
                                             <span slot="title">
                                                    <p v-html="item.title"></p>
                                             </span>
                                            <a-avatar style="background-color: white" slot="avatar"
                                                      src="https://gw.alipayobjects.com/zos/rmsportal/ThXAXghbEsBCCSDihZxY.png"/>
                                        </a-list-item-meta>
                                    </a-list-item>
                                </template>
                            </a-list>
                            <div class="footer-action">
                                <a class="item muted" @click="setRead('notice')">清空通知</a>
                                <a class="item muted" @click="()=>{$router.push('/notify/notice')}">查看更多</a>
                            </div>
                        </template>
                        <template v-else>
                            <div class="notFound">
                                <img src="../../../assets/image/notify/bell.svg" alt="not found">
                                <div>你已查看所有通知</div>
                            </div>
                        </template>
                    </a-tab-pane>
                    <a-tab-pane key="2">
                        <span slot="tab">消息<span
                                v-if="total && totalSum['message']">({{totalSum['message']}})</span></span>
                        <template v-if="total && totalSum['message']">
                            <a-list>
                                <template v-for="item in list['message']">
                                    <a-list-item>
                                        <a-list-item-meta :description="item.create_time">
                                             <span slot="title">
                                                    <p>{{item.title}}</p>
                                                    <p class="ant-list-item-meta-description" v-html="item.content"></p>
                                             </span>
                                            <a-avatar style="background-color: white" slot="avatar" v-if="item.from"
                                                      :src="item.from.avatar"/>
                                            <a-avatar style="background-color: white" slot="avatar" v-else
                                                      src="https://gw.alipayobjects.com/zos/rmsportal/OKJXDXrmkNshAMvwtvhu.png"/>
                                        </a-list-item-meta>
                                    </a-list-item>
                                </template>
                            </a-list>
                            <div class="footer-action">
                                <a class="item muted" @click="setRead('message')">清空消息</a>
                                <a class="item muted" @click="()=>{$router.push('/notify/message')}">查看更多</a>
                            </div>
                        </template>
                        <template v-else>
                            <div class="notFound">
                                <img src="../../../assets/image/notify/laba.svg" alt="not found">
                                <div>你已读完所有消息</div>
                            </div>
                        </template>
                    </a-tab-pane>
                    <a-tab-pane key="3">
                        <span slot="tab">待办<span
                                v-if="total && totalSum['task']">({{totalSum['task']}})</span></span>
                        <template v-if="total && totalSum['task']">
                            <a-list>
                                <template v-for="item in list['task']">
                                    <a-list-item>
                                        <a-list-item-meta :description="item.content">
                                             <span slot="title">
                                                    <p>{{item.title}}</p>
                                             </span>
                                        </a-list-item-meta>
                                    </a-list-item>
                                </template>
                            </a-list>
                            <div class="footer-action">
                                <a class="item muted" @click="setRead('task')">清空待办</a>
                                <a class="item muted">查看更多</a>
                            </div>
                        </template>
                        <template v-else>
                            <div class="notFound">
                                <img src="../../../assets/image/notify/ticket.svg" alt="not found">
                                <div>你已完成所有待办</div>
                            </div>
                        </template>
                    </a-tab-pane>
                </a-tabs>
            </a-spin>
        </template>
        <span>
          <a-badge :count="total">
            <a-icon class="action-item" type="bell"/>
          </a-badge>
        </span>
    </a-popover>
</template>

<script>
    import {mapState} from 'vuex'
    import {noReads} from "../../../api/notify";
    import {notice} from "../../../assets/js/notice";

    export default {
        name: 'HeaderNotice',
        data() {
            return {
                loadding: false,
                total: 0,
                totalSum: 0,
                list: []
            }
        },
        computed: {
            ...mapState({
                socketAction: state => state.socketAction,
                currentOrganization: state => state.currentOrganization,
            })
        },
        watch: {
            socketAction(val) {
                if (val.action === 'notice') {
                    this.fetchNotice();
                } else if (val.action === 'booking-create' || val.action === 'booking-checkout' || val.action === 'booking-cancel' || val.action === 'booking-allocating') {
                    console.log(val.data.organizationCode);
                    console.log(this.currentOrganization.code);
                    if (val.data.data.organizationCode != this.currentOrganization.code) {
                        return false;
                    }
                    if (val.action === 'booking-create') {
                        autoPlay('order2');
                    }else{
                        autoPlay('qianzou');
                    }
                    this.fetchNotice();
                    notice(val, 'notice', 'info', 100, 'bottomLeft')
                }
            }
        },
        created() {
            this.fetchNotice();
        },
        methods: {
            fetchNotice() {
                let app = this;
                noReads().then(res => {
                    app.list = res.data.list;
                    app.total = res.data.total;
                    app.totalSum = res.data.totalSum;
                });
            },
            setRead(type) {
                this.total -= this.list[type].length;
                this.list[type] = [];
            }
        }
    }
</script>

<style lang="less">
    .header-notice-content {
        width: 300px;
        .ant-tabs-bar {
            text-align: center;
            margin-bottom: 5px;
        }
        .ant-list-item-meta-title {
            p {
                margin-bottom: 8px;
            }
        }
        .ant-list-item-meta-description {
            font-size: 12px;
        }
        .ant-list-item:hover {
            /*background: #e6f7ff;*/
            cursor: pointer;
        }
        .notFound {
            text-align: center;
            padding: 73px 0 88px 0;
            color: rgba(0, 0, 0, 0.45);
            height: 275px;
            img {
                margin-bottom: 16px;
            }
        }
        .footer-action {
            border-top: 1px solid #e8e8e8;
            padding: 12px 0 0 0;
            .item {
                width: 49%;
                display: inline-block;
                text-align: center;
            }
        }
    }
</style>
