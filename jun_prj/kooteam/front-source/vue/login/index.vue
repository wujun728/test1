<template>
    <div class="bl inline-block" @click="show">
        <slot></slot>
        <z-dialog width="450px" :title="title" v-if="visible" visible @close="close">
            <QR v-if="showQr" v-model="params"></QR>
            <Password v-if="!showQr" v-model="params" :target="target"></Password>
            <div class="ft hover blue" style="text-align: right" v-if="isQr" @click="change">
                {{tip}}
            </div>
        </z-dialog>
    </div>
</template>
<script>
    import Password from './password';
    import QR from './qr';

    export default {
        components: {Password, QR},
        data: function () {
            return {
                isQr: false,
                showQr: false,
                visible: false,
                params: null,
                target: '/todo/home.xhtm',
                timerId: null
            };
        },
        computed: {
            tip() {
                return this.showQr ? '账号密码登陆' : '手机扫码登陆';
            },
            title() {
                let params = this.params;
                if (!params || !this.isQr) {
                    return '用户登录';
                }
                if (params.type === 'dingding') {
                    return '请用钉钉扫码登录';
                }
                if (params.type === 'wechat') {
                    return '请用企业微信扫码登录';
                }
                if (params.type === 'cloud') {
                    return '微信扫码,关注后登录';
                }
                if (params.type === 'ladp') {
                    return '请使用域账号登录';
                }
                return '用户登录';
            }
        },
        async mounted() {
            let data = await $.get(null, '/home/loginCheck.do');
            this.params = data;
            if (!data || !data.type || data.type === "ladp") {
                return;
            }
            this.isQr = true;
            this.showQr = true;
            if (data.type === "dingding" || data.type === "cloud") {
                this.timerId = setInterval(this.checkId, 2000);
            }
        },
        beforeDestroy() {
            clearInterval(this.timerId);
        },
        methods: {
            show() {
                this.visible = true;
            },
            close() {
                this.visible = false;
            },
            change() {
                this.showQr = !this.showQr;
            },
            checkId() {
                let id = this.params.checkId;
                if (!id) {
                    return;
                }
                $.get({checkId: id}, '/home/loginCheck.do', (reback) => {
                    let action = reback.action, data = reback.data;
                    if (action === 2) {
                        data.profile && $.cookie('zenUser', JSON.stringify(data.profile))
                        data.uri && $.goto(data.uri)
                    }
                });
            }
        }
    };
</script>