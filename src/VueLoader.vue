<template>
    <div :transition="transition" :class="transition">
        <slot></slot>
    </div>
</template>


/*!
* vue-loader v0.0.1 (https://github.com/johnnyGoo/vue-loader)
* Author: Johnny chen
*
* Copyright 2013-2015 Johnny chen
*/
<script>


    var count = 0;

    if (!window.Smart) {
        throw 'vue-loader required smart.js'
    }
    var Smart = window.Smart;
    var Css = Smart.Css;
    var Loader = Smart.Loader;
    var _ = Smart._;


    // 注册
    export default {
        // 声明 props
        props: {
            normal: {
                type: Object,
                default: function () {
                    return {opacity: 1,transition: 'all 0.5s ease'}
                }
            },
            enter: {
                type: Object,
                default: function () {
                    return {opacity: 0}
                }
            }, leave: {
                type: Object,
                default: function () {
                    return {opacity: 0}
                }

            }, assets: {
                default: null
            }, minDuration: {
                type: Number,
                default: 500
            }
        },
        data: function () {
            count++;
            return {
                transition: 'vue-loading-transition-count' + count,
                startTime: 0

            }
        },

        methods: {
            maxPercent: function () {
                return (new Date().getTime() - this.startTime) / this.minDuration
            },
            loadingComplete: function () {
                var me = this;
                me.$emit('loading-complete', me.loader);
                clearInterval(me.iv);
            },
            startLoading: function () {
                var me = this;
                me.startTime = new Date().getTime();

                me.loader = new Loader();

                var assets;
                if(Smart.Utils.isArray(me.assets)){
                    assets=(me.assets);
                }else{
                    assets=(me.loader.checkAllImages(me.assets));
                  //  console.log(me.loader.checkAllImages(me.assets))
                }
                me.loader.addImages(assets);
               // console.log(assets);
                me.$emit('loading-start');
                //监听资源加载进度事件
                me.loader.addProgressListener(function (p) {
                    me.$emit('loading-progress', {
                        completedCount: p.completedCount,
                        percent: Math.min(p.completedCount / p.totalCount, me.maxPercent()),
                        totalCount: p.totalCount,
                        img:p.img
                    });
                });
                
                
                function realloaded() {
                    me.iv = setInterval(function () {
                        me.$emit('loading-progress', {percent: me.maxPercent()});
                        if (me.maxPercent() >= 1) {
                            me.loadingComplete()
                        }
                    }, 30);
                }


                //监听资源加载完成事件
                me.loader.addCompletionListener(realloaded);
                //启动资源加载管理器
                me.loader.start();
                if(assets.length<=0){
                  //  realloaded()
                  _.delay(realloaded,10);
                }






            }
        },
        computed: {

        },


        ready: function () {
            if (_.isEqual({}, this.enter) && _.isEqual({}, this.leave)) {
                Css.smartCss(this.$el, this.normal, 'px');
                return;
            }
            if (_.isEqual({}, this.enter)) {
                this.enter = _.clone(this.leave);
            } else if (_.isEqual({}, this.leave)) {
                this.leave = _.clone(this.enter);
            }
            Css.createSmartCssStyle('.' + this.transition + '-transition', _.clone(this.normal), 'px');
            Css.createSmartCssStyle('.' + this.transition + '-leave', _.clone(this.leave), 'px');
            Css.createSmartCssStyle('.' + this.transition + '-enter', _.clone(this.enter), 'px');
            this.startLoading()
        }


    }


</script>