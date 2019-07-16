<!--我的页面-->
<template>
    <div id="home">
        <!--<v-head  goBack="true"></v-head>-->
        <div class="userTopWrapper"></div>
        <div id="user-info">
            <div class="avatar">
                <img :src="avatar">
                <input id="file" type="file" @change="fileUpload($event)" style="display: none;">
            </div>
            <router-link v-if="!username" class="login" to="/login">登录/注册</router-link>
            <span v-else class="username">{{username}}</span>
        </div>
        <div class="homeContent">
            <div class="userWrapper">
                <ul class="interestingContainer">
                    <li v-for="(item,index) in interestings" :key="index" @click="routerChange(item.url)">
                        <div class="imgWrapper"><img :src="item.picUrl"></div>
                        <span>{{item.name}}</span>
                    </li>
                </ul>
            </div>
            <div class="assets">
                <h4 class="title">我的资产</h4>
                <ul>
                    <li v-for="(item,index) in myAssetsList" :key="index" @click="routerChange(item.url)">
                        <div class="img">
                            <img :src="item.picUrl">
                        </div>
                        <span>{{item.name}}</span>
                    </li>
                </ul>
            </div>
            <div class="intro">
                <h4>更多推荐</h4>
                <ul>
                    <li v-for="(item,index) in introList" :key="index">
                        <div class="img">
                            <img :src="item.picUrl">
                        </div>
                        <span>{{item.name}}</span>
                    </li>
                </ul>
            </div>
            <alert-tip :text="alertText" :showTip.sync="showTip"></alert-tip>
        </div>
        <v-bottom v-show="ishome"></v-bottom>
        <router-view></router-view>
        <v-loading v-show="loading"></v-loading>
    </div>
</template>
<script>
    import {userInfo, changeAvatar} from '@/api/user'
    import {getInfo} from '@/utils/auth'
    import {uploadToken, upload} from '@/api/upload'
    import config from '@/config'
    import like from '@/assets/like.png'
    import comment from '@/assets/comment.png'
    import history from '@/assets/history.png'

    export default {
        data() {
            return {
                username: null,
                avatar: 'http://i.waimai.meituan.com/static/img/default-avatar.png',
                loading: false,
                alertText: '',
                showTip: false,
                ishome:false,
                interestings: [
                    {
                        url: '/home/collection',
                        picUrl: like,
                        name: '收藏'
                    },
                    {
                        url: '/home/comment',
                        picUrl: comment,
                        name: '评价'
                    },
                    {
                        url: '/home/footprint',
                        picUrl: history,
                        name: '足迹'
                    },
                ],
                myAssetsList:
                    [
                        {
                            url: '/home/friend',
                            picUrl: 'http://p1.meituan.net/50.0.100/xianfu/bbae84a587711ac12badb9453406ad694851.jpg',
                            name: '我的好友'
                        },
                        {
                            url: '/home/thank',
                            picUrl: 'http://p1.meituan.net/50.0.100/xianfu/5c1bf832376403ca2ab22b8d8748e0fd5479.jpg',
                            name: '答谢记录'
                        },
                        {
                            url: '/home/address',
                            picUrl: 'http://p0.meituan.net/50.0.100/xianfu/a813bff1813024b05ff45422deac24bd4276.jpg',
                            name: '我的地址'
                        },
                        {
                            url: '/home/hongbao',
                            name: '红包',
                            picUrl: 'http://p1.meituan.net/50.0.100/xianfu/a361ce97f9f00f2715bb960a789d925e2315.jpg',
                        },
                        {
                            url: '/home/friend',
                            name: '代金券',
                            picUrl: 'http://p0.meituan.net/50.0.100/xianfu/875f13a76045b7f6862a2b7149babec32329.jpg',
                        },
                        {
                            url: '/home/friend',
                            name: '钱包',
                            picUrl: 'http://p1.meituan.net/50.0.100/xianfu/2c14b3425c7bf1f3d63d11f47a7ef9ea2230.jpg',
                        },
                        {
                            url: '/home/amount',
                            name: '余额',
                            picUrl: 'http://p0.meituan.net/50.0.100/xianfu/7b3e3fb4fc9b45dcda74d7e916f025ea2878.jpg'
                        }
                    ],
                introList:
                    [
                        {
                            picUrl: 'http://p0.meituan.net/50.0.100/xianfu/cf5ddfcae114ed8d7d147d51064532252477.jpg',
                            name: '邀请有奖'
                        },
                        {
                            picUrl: 'http://p1.meituan.net/50.0.100/xianfu/55748d5fa531a057258f68d029fe20542466.jpg',
                            name: '商家入驻'
                        },
                        {
                            picUrl: 'http://p1.meituan.net/50.0.100/xianfu/317aabdd31dfcfa1739149089a2e041a2780.jpg',
                            name: '帮助与反馈'
                        },
                        {
                            picUrl: 'http://p0.meituan.net/50.0.100/xianfu/55454d4faaed6ad212b2b8a929edef372425.jpg',
                            name: '在线客服'
                        },
                    ]
            }
        },
        methods: {
            fileUpload(event) {
                this.loading = true;
                let file = event.target.files[0];
                if (file.size > 1024 * 1024 * 3) {    //只能传2M以内照片
                    this.alertText = '上传失败，只能传2M以内图片'
                    this.showTip = true;
                } else {
                    uploadToken().then((response) => {
                        if (response.data.status === 200) {
                            let data = {token: response.data.uptoken, file}
                            upload(data).then((upResponse) => {
                                let pic_url = config.domain + upResponse.data.key
                                this.avatar = pic_url;
                                this.loading = false;
                                changeAvatar({pic_url}).then(() => {
                                })     //更新到数据库
                            })
                        } else {
                            this.alertText = response.data.message
                            this.showTip = true;
                        }
                    })
                }
            },
            routerChange(url) {
                this.ishome=false;
                if (this.username) {
                    this.$router.push(url);
                } else {
                    this.$router.push('/login');
                }
            }
        },
        mounted() {
            this.username = getInfo();
            if (this.username) {
                userInfo().then((response) => {
                    this.avatar = response.data.data.avatar;
                })
            }
        }
    }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
    @import "../../style/common.scss";
    @import "../../style/mixin.scss";
    #home{
        width: 100%;
        height: 100%;
        overflow: hidden;
    }

    .userTopWrapper {
        @include px2rem(height, 250);
        background: linear-gradient(to top, white, cyan);
        width: 100%;
    }

    #user-info {
        background: white;
        width: 120%;
        @include px2rem(height, 300);
        color: #000;
        margin-left: -1rem;
        margin-top: -1rem;
        text-align: center;
        /*display: flex;*/
        /*align-items: center;*/
        border: 1px solid white;
        border-radius: 70%;

        .avatar {
            @include px2rem(width, 115);
            @include px2rem(height, 115);
            display: block;
            margin: -0.7rem auto 0;
            border-radius: 50%;
            overflow: hidden;
            border: 5px solid #fafafa;
            img {
                width: 100%;
                height: 100%;
            }
        }

        .login {
            font-size: 0.45rem;
        }

        .username {
            font-size: 0.5rem;
        }
    }

    .homeContent {
        padding: 0 0.3rem;
    }

    .userWrapper {
        padding: 0 0.4rem;
        background: white;
        @include px2rem(height, 150);
        margin-top: -1rem;
        width: 100%;

        .interestingContainer {
            border-bottom: #f5f5f5 1px solid;
            display: flex;
            justify-content: center;
            align-items: center;
            padding-bottom: 0.4rem;

            .imgWrapper {
                margin-bottom: 0.2rem;
            }

            li {
                margin-bottom: 0.2rem;
                text-align: center;
                width: 33%;
                cursor:pointer;
                img {
                    width: 25%;
                }
                img:hover{
                    cursor: pointer;
                }
            }

        }
    }

    .fun-container {
        margin-top: 0.3rem;
        background: #fff;

        ul {
            @include clearfix;

            li {
                width: 25%;
                @include px2rem(height, 145);
                float: left;
                text-align: center;
                margin: 0.2rem 0;

                .img {
                    @include px2rem(width, 70);
                    @include px2rem(height, 70);
                    margin: 0.1rem auto;

                    img {
                        width: 100%;
                        height: 100%;
                    }
                }

                span {
                    font-size: 0.35rem;
                }
            }
        }
    }

    .assets, .intro {
        margin-top: 0.2rem;
        background: #fff;
        padding: 0 0.3rem;

        h4 {
            font-size: 0.3rem;
            font-weight: bold;
            padding-bottom: 0.2rem;
        }

        ul {
            display: flex;
            flex-wrap: wrap;
            padding: 0.3rem 0;
            li {
                text-align: center;
                margin-bottom: 0.3rem;
                width: 25%;
                .img {
                    @include px2rem(width, 55);
                    @include px2rem(height, 55);
                    margin: 0.1rem auto;
                    img {
                        width: 100%;
                        height: 100%;
                    }
                }

                span {
                    font-size: 0.35rem;
                }
            }
        }
    }

    .intro {
        padding-bottom: 1rem;
    }
</style>
