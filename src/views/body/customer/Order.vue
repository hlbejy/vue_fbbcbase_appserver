<template>
    <div class="content" ref="scrollContainer">
        <div class="account-ds">
            <div class="bar bar-nav account-top-m">
                <router-link to="/customer/account/index"  class="button button-link button-nav pull-left">
                    <span class="icon icon-left"></span>
                </router-link>
                <h1 class='title'>
                    {{ $t("message.customer_order") }}
                </h1>
            </div>
        </div>
        <div class="fecshop_message" v-if="errormsg">
            <div class="error-msg">
                <div>
                    {{errormsg}}
                </div>
            </div>
		</div>
        <div class="order_list">
            <div class="col-main account_center customer_address_list">
                <div class="std">
                    <div style="margin:4px 0 0">
                    
                        <div class="account_customer_address" id="billing_address" v-for="(item,index) in orderList">	
                            <ul>
                                <li>
                                    <div>
                                        <p>
                                            {{ $t("message.order_increment_id") }}：{{item.increment_id}}
                                            
                                            <span style="display:block;float:right">{{item.created_at}}</span>
                                        </p>
                                        <p style="color:#777;">
                                             <span v-if="item.order_status">{{ $t("message.order_status") }}：{{item.order_status}}<br/></span>
                                             <span v-if="item.shipping_method">{{ $t("message.shipping_method") }}：{{item.shipping_method}}<br/></span>
                                             <span v-if="item.payment_method">{{ $t("message.payment_method") }}：{{item.payment_method}}<br/></span>
                                            <span v-if="item.tracking_number">{{ $t("message.tracking_number") }}：{{item.tracking_number}}</span>
                                        </p>
                                        <p class="order_operate_it">
                                            <span class="nobr"><a @click="viewOrder(item.order_id)"  href="javascript:void(0)">{{ $t("message.view") }}</a>
                                            <span v-if="item.can_payment" class="separator ">|</span> <a v-if="item.can_payment" @click="paymentOrder(item.increment_id)" class="link-payment-order" href="javascript:void(0)" >{{ $t("message.payment") }}</a>
                                            <span v-if="item.can_cancel" class="separator">|</span> <a v-if="item.can_cancel" @click="cancelOrder(item.increment_id)" class="link-cancelorder" href="javascript:void(0)" >{{ $t("message.cancel") }}</a>
                                            <span v-if="item.can_receive" class="separator">|</span> <a v-if="item.can_receive" @click="receiveOrder(item.increment_id)" class="link-receiveorder" href="javascript:void(0)" >{{ $t("message.receive_order") }}</a>
                                            <span v-if="item.can_after_sale" class="separator">|</span> <a v-if="item.can_after_sale" @click="afterSaleOrder(item.order_id)" class="link-aftersaleorder" href="javascript:void(0)" >{{ $t("message.after_sale_order") }}</a>
                                            <span v-if="item.can_cancel_back" class="separator">|</span> <a v-if="item.can_cancel_back" @click="cancelBackOrder(item.increment_id)" class="link-aftersaleorder" href="javascript:void(0)" >{{ $t("message.cancel_back") }}</a>
                                            <span v-if="item.can_delay_receive" class="separator">|</span> <a v-if="item.can_delay_receive" @click="delayReceiveOrder(item.order_id)" class="link-aftersaleorder" href="javascript:void(0)" >{{ $t("message.delay_receive") }}</a>
                                            
                                            
                                        </p>
                                    </div>
                                    
                                    
                                </li>
                            </ul>
                        </div>
                        
                    
                        <!-- 加载提示符 -->
                        <mugen-scroll :handler="fetchOrder" :should-handle="!loading" scroll-container="scrollContainer">
                            <div style="display:none;"  class="infinite-scroll-preloader">
                                <div class="preloader"></div>
                            </div>
                        </mugen-scroll>
                    </div>
                </div>
            </div>
            <div class="clear"></div>
        </div>
    </div>
</template>
<script>
import MugenScroll from 'vue-mugen-scroll'
var root = process.env.API_ROOT;

export default {
    data () {
        return {
            pageInitUrl: root + '/customer/order/index' ,
            cancelOrderUrl: root + '/customer/order/cancel' ,
            receiveOrderUrl: root + '/customer/order/receive' ,
            afterSaleOrderUrl: root + '/customer/order/aftersale' ,
            cancelBackOrderUrl: root + '/customer/order/cancelback' ,
            delayReceiveOrderUrl: root + '/customer/order/delayreceive' ,
            errormsg:'',
            orderList:[],
            count:0,
            loading: false ,
            refer_url: '',
            correctmsg:''
        }
    },
    created: function(){
        this.pageInit();
    },
    components:{
        'mugen-scroll': MugenScroll
    },
    beforeRouteEnter (to, from, next) {
        var website_root = process.env.WEBSITE_ROOT
        var fullPath = from.fullPath
        var name = from.name
        console.log(fullPath);
        console.log(from);  
        if (fullPath !== '/' || typeof(name) === 'undefined' ) {
            var referUrl = website_root + "/#" + fullPath
            console.log(referUrl)
            
        } else {
            referUrl = ''
        }
        next( vm => {
            vm.refer_url = referUrl;
        });  
    },
    methods: {
        pageInit: function(){
            var self = this;
            self.errormsg = '';
            self.correctmsg = '';
            $.showIndicator();
            $.ajax({
                url: self.pageInitUrl,
                async: true,
                timeout: 120000,
                type: 'get',
                headers: self.getRequestHeader(),
                data:{ 
                },
                success:function(reponseData, textStatus,request){
                    $.hideIndicator();
                    if(reponseData.code == 1100003){
                        self.$router.push('/customer/account/login');
                        return;
                    }else if(reponseData.code == 200){
                        self.orderList = reponseData.data.orderList;
                        console.log('get customer order info success');
                        var traceData = {"refer_url": self.refer_url};
                        var routerQ = self.$route.query
                        for (var k in routerQ) {
                            traceData[k] = routerQ[k]
                        }
                        self.reloadTraceJs(traceData);
                        self.saveReponseHeader(request); 
                        self.count = 1;
                    }
                    
                },
                error:function(){
                    $.toast(self.$i18n.t("message.system_error"));
                    $.hideIndicator();
                    console.log('get customer order info error');
                }
            });
        },
        fetchOrder: function(){
            var self = this;
            if(self.count >=1){
                self.loading = true;
                self.errormsg = '';
                self.correctmsg = '';
                $.showIndicator();
                $.ajax({
                    url: self.pageInitUrl,
                    async: true,
                    timeout: 120000,
                    type: 'get',
                    headers: self.getRequestHeader(),
                    data:{ 
                        p: self.count+1
                    },
                    success:function(reponseData, textStatus,request){
                        $.hideIndicator();
                        if(reponseData.code == 1100003){
                            self.$router.push('/customer/account/login');
                            return;
                        }else if(reponseData.code == 200){
                            console.log('fetch product 200');
                            var orders = reponseData.data.orderList;
                            if(orders.length > 0){
                                for(var x in orders){
                                    self.orderList.push(orders[x]);
                                }
                                self.loading = false;
                                console.log('fetch order loading false');
                                $.init();
                                self.count++;
                            }
                            self.saveReponseHeader(request);
                        }
                        
                    },
                    error:function(){
                        $.toast(self.$i18n.t("message.system_error"));
                        $.hideIndicator();
                        console.log('get customer order info error');
                    }
                });
            }
        },
        viewOrder :function(order_id){
            var self = this;
            self.$router.push('/customer/order/view/'+order_id);
        },
        paymentOrder :function(order_increment_id){
            var self = this;
            self.$router.push('/checkout/payment/' + order_increment_id);
        },
        cancelOrder :function(order_increment_id){
            var self = this;
            self.loading = true;
            self.errormsg = '';
            self.correctmsg = '';
            $.showIndicator();
            $.ajax({
                url: self.cancelOrderUrl,
                async: true,
                timeout: 120000,
                type: 'get',
                headers: self.getRequestHeader(),
                data:{ 
                    increment_id: order_increment_id
                },
                success:function(reponseData, textStatus,request){
                    $.hideIndicator();
                    if(reponseData.code == 1100003){
                        self.$router.push('/customer/account/login');
                        return;
                    }else if(reponseData.code == 200){
                        console.log('fetch product 200');
                        $.toast(self.$i18n.t("message.order_cancel_request_success"));
                        self.saveReponseHeader(request);
                        self.pageInit();
                    } else {
                        $.toast(self.$i18n.t("message.order_cancel_request_fail"));
                    }
                },
                error:function(){
                    $.toast(self.$i18n.t("message.system_error"));
                    $.hideIndicator();
                    console.log('get customer order info error');
                }
            });
        },
        receiveOrder :function(order_increment_id){
            var self = this;
            self.loading = true;
            self.errormsg = '';
            self.correctmsg = '';
            $.showIndicator();
            $.ajax({
                url: self.receiveOrderUrl,
                async: true,
                timeout: 120000,
                type: 'get',
                headers: self.getRequestHeader(),
                data:{ 
                    increment_id: order_increment_id
                },
                success:function(reponseData, textStatus,request){
                    $.hideIndicator();
                    if(reponseData.code == 1100003){
                        self.$router.push('/customer/account/login');
                        return;
                    }else if(reponseData.code == 200){
                        console.log('fetch product 200');
                        $.toast(self.$i18n.t("message.order_receive_request_success"));
                        self.saveReponseHeader(request);
                        self.pageInit();
                    } else {
                        $.toast(self.$i18n.t("message.order_receive_request_fail"));
                    }
                },
                error:function(){
                    $.toast(self.$i18n.t("message.system_error"));
                    $.hideIndicator();
                    console.log('get customer order info error');
                }
            });
        },
        afterSaleOrder :function(order_increment_id){
            var self = this;
            self.$router.push('/customer/order/aftersale/'+order_increment_id);
        },
        
        delayReceiveOrder :function(order_id){
            var self = this;
            self.loading = true;
            self.errormsg = '';
            self.correctmsg = '';
            $.showIndicator();
            $.ajax({
                url: self.delayReceiveOrderUrl,
                async: true,
                timeout: 120000,
                type: 'get',
                headers: self.getRequestHeader(),
                data:{ 
                    order_id: order_id
                },
                success:function(reponseData, textStatus,request){
                    $.hideIndicator();
                    if(reponseData.code == 1100003){
                        self.$router.push('/customer/account/login');
                        return;
                    }else if(reponseData.code == 200){
                        console.log('fetch 200');
                        $.toast(self.$i18n.t("message.order_delay_receive_request_success"));
                        self.saveReponseHeader(request);
                        self.pageInit();
                    } else {
                        $.toast(self.$i18n.t("message.order_delay_receive_request_fail"));
                    }
                },
                error:function(){
                    $.toast(self.$i18n.t("message.system_error"));
                    $.hideIndicator();
                    console.log('get customer order info error');
                }
            });
        },
        cancelBackOrder :function(order_increment_id){
            var self = this;
            self.loading = true;
            self.errormsg = '';
            self.correctmsg = '';
            $.showIndicator();
            $.ajax({
                url: self.cancelBackOrderUrl,
                async: true,
                timeout: 120000,
                type: 'get',
                headers: self.getRequestHeader(),
                data:{ 
                    increment_id: order_increment_id
                },
                success:function(reponseData, textStatus,request){
                    $.hideIndicator();
                    if(reponseData.code == 1100003){
                        self.$router.push('/customer/account/login');
                        return;
                    }else if(reponseData.code == 200){
                        console.log('fetch 200');
                        $.toast(self.$i18n.t("message.order_cancel_back_request_success"));
                        self.saveReponseHeader(request);
                        self.pageInit();
                    } else {
                        $.toast(self.$i18n.t("message.order_cancel_back_request_fail"));
                    }
                },
                error:function(){
                    $.toast(self.$i18n.t("message.system_error"));
                    $.hideIndicator();
                    console.log('get customer order info error');
                }
            });
        }
        
    }
    
    
    
}
</script>