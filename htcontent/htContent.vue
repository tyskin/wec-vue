<template>
    <div class="deal-content">
        <div class="deal-content-header bh-clearfix" :class="{noDealHeader:noHeader}">
            <div class="content-header-left bh-pull-left">
                <span class='htdg vmiddle'>合同大纲</span>
                <i class="iconfont icon-help bh-color-primary vmiddle"></i>
                <bh-checkbox :ischeck='false' class="content-header-check bh-pull-right" @change="filter">
                    <span v-if="needAudit">只看我审核的</span>
                    <span v-else>只看已退回的</span>
                </bh-checkbox>
            </div>
            <div class="deal-header-right bh-pull-left">
                <span>合同内容</span>
            </div>
        </div>
        <div class="deal-content-body bh-clearfix">
            <div class="content-body-left">
                <outline v-ref:outline :wid='wid' @select="outlineSelect" :url="outlineUrl"></outline>
            </div>
            <div class="content-body-right">
                <div class="content-entry-con">
                    <entry-con v-ref:ec :wid='wid' @ready="getTop" :url="entryUrl" :flow-url='auditUrl' :not-review="notReview"></entry-con>
                </div>
            </div>
        </div>
    </div>
</template>

<script type="text/ecmascript-6">
    /**
     * @module dealcontent
     * 合同内容
     *
     *
     * @example
     * <deal-content :outline-url="outlineUrl" :entry-url="url" :need-audit="true"></deal-content>
     *
     * @example
     * export default {
     *   data: () => ({
     *     needAudit: 是否查看需要审核的,
     *     outlineUrl:获取大纲的url
     *     entryUrl: 获取右侧的条目的数据,
     *     notReview:是否需要展示右侧的审核
     *   })
     * }
     */
    import BhCheckbox from 'bh-vue/bh-checkbox/bhCheckbox.vue';
    import pageUtil from 'bh-vue/utils/pageUtil';
    import outline from './outline.vue';
    import entryCon from './entryCon.vue';
    import pop from './pop';

    export default {
        data: () => {
        return {
            isAudit: false,
            hasChecked:false
        }
    },
    props: {
        dealContent: Array,
        needAudit: Boolean,
        entryUrl: String,
        outlineUrl: String,
        auditUrl: String, // 查询审核进度
        wid: String, // 合同 id
        mbId: String, // 合同模板 id
        notReview:Boolean,
        noHeader:Boolean
    },
    ready(){
        let vm = this
        $('.outline-item').eq(0).addClass('outline-active')
        let entryHeight = $('article').attr('style').replace(/[^0-9]/ig, "");
        $(".content-entry-con").css('height', parseInt(entryHeight - 200) + 'px')
        $(".content-entry-con").niceScroll()
        $('.content-header-left').on('mouseover', '.iconfont', (e) => {
            pageUtil.showPopover(e.target, '<pop></pop>', {
                showCloseButton: false,
                autoClose: true,
                height: '32px',
                width: '230px',
                ready: function (popWindow) {
                    vm.$compile(popWindow[0])
                }
            })
        })
        $('.content-header-left').on('mouseleave', '.iconfont', (e) => {
            pageUtil.hidePopover();
        })
    },
    methods: {
        outlineSelect(item){
            let $title = $(event.currentTarget)
            let conTop = $(".content-entry-con").offset().top
            let itemOffsetTop = this.map['entry' + item.wid]
            let fixedTop = itemOffsetTop - conTop - 10
            $('.outline-active').removeClass('outline-active')
            $title.addClass('outline-active')
            $(".content-entry-con").animate({
                scrollTop: (fixedTop)
            }, 450);
        },
        getTop(item){
            this.map = {}
            let entryArray = $('.entry-con')
            for (let i = 0; i < entryArray.length; i++) {
                let $dom = entryArray.eq(i)
                let wid = $dom.attr('data-wid')
                this.map['entry' + wid] = $dom.offset().top
            }
        },
        filter(e){
            if (e.checked) {
                this.hasChecked=true
                if (this.needAudit) {
                    this.isAudit = true
                }
            } else {
                this.hasChecked=false
            }

            this.$refs.outline.update(this.isAudit,this.hasChecked)
        },
        getCount () {
            return this.$refs.ec.getCount();
        },
        getTermList () {
            return this.$refs.ec.getTermList();
        }
    },
    components: {BhCheckbox, outline, entryCon, pop}
    };
</script>

<style scoped>
    .vmiddle {
        vertical-align: middle;
        cursor: pointer;
    }

    .htdg {
        line-height: 30px;
    }

    .deal-content-header {
        background-color: #F7F8FC;
        border-left: 1px solid #D8DCF0;
        border-right: 1px solid #D8DCF0;
        border-top: 1px solid #D8DCF0;
        height: 32px;
    }

    .deal-content-body {
        border: 1px solid #D8DCF0;
        display: flex;
    }

    .content-body-left {
        border-right: 1px solid #D8DCF0;
        width: 190px;
        display: inline-block;
        float: left;
    }

    .content-header-left {
        border-right: 1px solid #D8DCF0;
        width: 190px;
        padding-left: 8px;
    }

    .content-header-left span,
    .deal-header-right span,
    .content-header-check span {
        color: #333;
        font-weight: 600;
    }

    .content-header-left .iconfont {
        line-height: 30px;
    }

    .content-header-check span {
        vertical-align: text-top !important;
        margin-left: -7px;
    }

    .deal-header-right {
        width: calc(100% - 190px);
        padding-left: 12px;
    }

    .content-body-right {
        width: calc(100% - 190px);
        float: left;
    }

    .content-header-check .bh-checkbox {
        padding-top: 2px;
    }

    .deal-header-right span {
        line-height: 30px;
    }

    .outline-active {
        border-left: 4px solid #2196f3;
        background-color: #D3EAFD;
    }

    .content-entry-con {
        overflow: hidden;
        padding-top: 10px;
    }
    .deal-content-header.noDealHeader{
        display: none;
    }

</style>
