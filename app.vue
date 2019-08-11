<template>
    <div class="layout-container content-box">
    111
    222
    333
    这是不能上线的东西
        <!--搜索条件筛选框 start-->
        <div class="project-list">
            <Tabs :animated="false" :value="activeTab" @on-click="changeTab">
                <TabPane :label="labelInvite" name="invite">
                    <yCustomSearch
                            class="mg-b10 mg-t20"
                            :isResetFlag="activeTab"
                        :templateId="this.GLOBAL.omsTemplateIdJSON.invite"
                        @getSearchData="getSearchData"
                    ></yCustomSearch>
                </TabPane>
                <TabPane :label="labelGoingQuote" name="goingQuote">
                    <yCustomSearch
                            class="mg-b10 mg-t20"
                            :isResetFlag="activeTab"
                            :templateId="this.GLOBAL.omsTemplateIdJSON.goingQuote"
                            @getSearchData="getSearchData"
                    ></yCustomSearch>
                </TabPane>
                <TabPane :label="labelDoneQuote" name="doneQuote">
                    <yCustomSearch
                            class="mg-b10 mg-t20"
                            :isResetFlag="activeTab"
                            :templateId="this.GLOBAL.omsTemplateIdJSON.doneQuote"
                            @getSearchData="getSearchData"
                    ></yCustomSearch>
                </TabPane>
            </Tabs>
            <!--调用列表渲染组件（考虑将所有列表渲染抽成一个公用组件，现因列宽度不能从后端取得，不同页列宽需单独定义，试通过数组传值实现或等后续接口再改造）-->
            <QuoteList ref="child" :searchJSON="searchJSON" @isEmptySearchVal="isEmptySearchVal"
                       :subTypeStatus="subTypeStatus"></QuoteList>
        </div>
    </div>
</template>
<script>
    import QuoteList from './quote-list.vue';
//    import yCustomSearch from 'yvue/src/components/y-custom-search'

    export default {
        components: {
            'QuoteList': QuoteList,
//            'yCustomSearch':yCustomSearch
        },
        data() {
            return {
                activeTab: '/',
                urlJSON: {
                    getBtnStatus: this.GLOBAL.dataUrl + '/trade/endTrade/findEndTradeStatus',//项目状态
                    invite: this.GLOBAL.dataUrl + '/trade/quotedPriceTrade/inviteTradeList', // 受邀项目
                    recommend: this.GLOBAL.dataUrl + '/trade/quotedPriceTrade/recommendTradeList',
                    canjoin: this.GLOBAL.dataUrl + '/trade/quotedPriceTrade/canjoinTradeList',

                    getTabNumber: this.GLOBAL.dataUrl + '/trade/quotedPriceTrade/gainTradeProjectCountMap', // 获取tab选项卡的数字

                    getTest: this.GLOBAL.dataUrl + '/trade/test'
                },
                domainStr: this.GLOBAL.domainStr,

                subTypeStatus: this.activeTab, // 列表转tab切换
                searchJSON:null, // 搜索条件
                searchType: null, // 查询，选择下拉
                pageData: {},
//                invite 受邀项目、goingQuote 进行之中、doneQuote 已经结束
                labelInviteTotal:0,
                labelGoingQuoteTotal:0,
                labelDoneQuoteTotal:0,
                labelInvite: (h) => {
                    return h('div', [
                        h('span', '受邀项目'),
                        h('Badge', {
                            class: {'custom-tabs-badge--default': true},
                            props: {
                                'overflow-count': '9999',
                                count: this.labelInviteTotal
                            }
                        }),
                    ]);
                },
                labelGoingQuote: (h) => {
                    return h('div', [
                        h('span', '进行之中'),
                        h('Badge', {
                            class: {'custom-tabs-badge--default': true},
                            props: {
                                'overflow-count': '9999',
                                count: this.labelGoingQuoteTotal
                            }
                        }),
                    ]);
                },
                labelDoneQuote: (h) => {
                    return h('div', [
                        h('span', '已经结束'),
                        h('Badge', {
                            class: {'custom-tabs-badge--default': true},
                            props: {
                                'overflow-count': '9999',
                                count: this.labelDoneQuoteTotal
                            }
                        }),
                    ]);
                },
            };
        },
        created() {
            this.fetchData();
        },
        watch: {
            // 如果路由有变化，会再次执行该方法
            '$route': 'fetchData',
        },
        mounted(){
            // 获取tab 选项卡的 数字
            this.getTabLabelNumber();
        },
        methods: {
            isEmptySearchVal(data) {
                this.pageData = data;
            },
            changeTab(name) {
                let _this = this;
                //Tab切换发生改变时清空搜索条件
                this.searchJSON={}; // 清空
                this.activeTab = name; // 高亮tab
                this.subTypeStatus = name;
                this.pageNum = 1;
                this.pageSize = 10;
                // 获取tab 选项卡的 数字
                this.getTabLabelNumber();
            },
            /**
             * 接收路由信息
             * 并更新面包屑、菜单
             */
            fetchData() {
//                this.subTypeStatus = this.$route.meta.typeStatus;
//                this.activeTab=this.$route.meta.typeStatus;
                this.activeTab = this.$route.params.typeStatus; // 获得tab名称并使之高亮
                this.subTypeStatus = this.$route.params.typeStatus; // 列表获得对应URL的参数
            },
            /**
             * 点击搜索框的按钮提交搜索的条件-刷新表格数据
             * 并更新面包屑、菜单
             */
            subSearchCondition() {
                //获取从子组件拿到的pageSize
                let _pageSize = this.pageData.pageSize;

                /**
                 * 判断pageSize是否存在
                 * ·页面一加载pageSize值不存在 设置一个默认参数【每页展示10条数据】
                 * ·存在则用已存在的pageSize值
                 */
                if (_pageSize === undefined) {
                    _pageSize = 10;
                }

                this.$refs.child.getInitTodoData({
                    tabName: this.activeTab,
                    pageNum: 1,
                    pageSize: _pageSize,
                    searchJSON:this.searchJSON
                })
            },
        },

    }
</script>