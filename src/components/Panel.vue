<template>
  <div id="panel" class='pa flex flex-column trans' :class='{"lt-white": ActiveView === "meta"}'>
    <div class='pa2 tc br-clr h-head' :class='{"br-none":list.details.length > 0}' >
      <div class='flex pa1'>
        <!-- <button class='btn btn-default btn-xs' v-if='ActiveView === "group"' @click='pushFilter'>
          <i class="fa fa-chevron-left f4" aria-hidden="true"></i>
        </button> -->
        
        <span class='w-50 f4' v-if='ActiveView !== "meta"'>
            {{ (ActiveView === 'details') ? ActiveMetaData.groupName : "" }}
            
           <i v-if='ActiveView === "details"' class="fa fa-chevron-right f5" aria-hidden="true"></i>
            {{ (ActiveView !== "meta") ?  activeMeta.groupName : "Billing Report" }}
            
        </span>
        <span class='w-30' v-if='ActiveView === "details" || ActiveView === "group"'>Total bills - 
          <span class='badge badge-primary' v-if='ActiveView === "details"'>
             {{ (DetailsList.length > 0 && 
               !DetailsList[0].hasOwnProperty('groupName')) ?
                                         DetailsList.length : 
                                         DetailsList.map(x => x.bills.length).reduce((a,b)=> a+b,0)
            }}
          </span>
          <span class='badge badge-primary' v-if='ActiveView === "group"'>
             {{     (list.group.length > 0) ? list.group.map(x => Number(x.noOfTransactions)).reduce((a,b)=> a+b,0) : 0
            }}
          </span>
        </span>
        <span class='w-20' v-if='(ActiveView === "group" || ActiveView === "details") && (list.group.length > 0 || DetailsList.length > 0)'>
          <button class='btn btn-xs btn-default' v-if='(ActiveView === "group" || ActiveView === "details") && (list.group.length > 0 || DetailsList.length > 0)' @click='getReport'>
           Export <i title='export this as report'  class="fa fa-download cursor" aria-hidden="true"></i>
          </button>
        </span>
        <!-- <span style='padding:1px;' v-if='ActiveView === "meta"'>Date Type</span> -->
        <!-- <span  class='_flx_full f4' v-if='false'>
          <span class="flex w-100 pa1">            
            <button class='btn btn-default btn-xs btn-ghost w-33' :class='{"btn-ghost-act" : DateFilter === "all"}' @click='DateFilter = "all"'>All</button>
            <button class='btn btn-default btn-xs btn-ghost w-33' :class='{"btn-ghost-act" : DateFilter === "booking"}' @click='DateFilter = "booking"'>Booking</button>
            <button class='btn btn-default btn-xs btn-ghost w-33' :class='{"btn-ghost-act" : DateFilter === "checkin"}' @click='DateFilter = "checkin"'>Checkin</button>
            <button class='btn btn-default btn-xs btn-ghost w-33' :class='{"btn-ghost-act" : DateFilter === "checkout"}' @click='DateFilter = "checkout"'>Checkout</button>
          </span>
        </span> -->
        <span class='pa1 tc w-100' v-if='ActiveView === "meta"'>Pick a date range</span>
      </div>
      <!--datepicker -->
      <div class='tc' v-if='ActiveView === "meta"'>
        <datepicker @input='setDate' calendar-class='w200' wrapper-class='di' input-class=' w40 btn btn-default btn-xs'	 v-model='filterDate.from'></datepicker> 
        <div class='di w20'>to</div>
        <datepicker  @input='setDate' calendar-class='w200' wrapper-class='di' input-class='w40 btn btn-default btn-xs'	 v-model='filterDate.to' :disabled-dates='{to: filterDate.from}'></datepicker>
      </div>
       <!-- Number of transction label -->
      <!-- <div class='tc' v-if='ActiveView === "details"'>
        
          <i class="fa fa-file-text-o" aria-hidden="true"></i> Number of transactions - 
          {{ (list.details.length > 0 && 
               !list.details[0].hasOwnProperty('groupName')) ?
                                         list.details.length : 
                                         (list.details.length > 0) ? list.details.map(x => x.bills.length).reduce((a,b)=> a+b) : 0
           }}

      </div> -->
      <!-- <div class='tc' v-if='ActiveView === "group"'>
        
          <i class="fa fa-file-text-o" aria-hidden="true"></i> Number of transactions - 
          {{  (list.group.length > 0) ? list.group.map(x => Number(x.noOfTransactions)).reduce((a,b)=> a+b) : 0 }}

      </div> -->
       <!-- search text box -->
      <div class='tl pa1' v-if='ActiveView === "group"'>
          <div class='br-clr w100 flex'> 
            <!-- <i class="fa fa-search dib" aria-hidden="true"></i> -->
            <input class='pa2 b5 w-100 br-white' type='text' v-model='searchString' @input='saySmthn' :placeholder="'Search in '+activeMeta.groupName">
          </div>
      </div>
      <!--- group filter -->
      <div class='tc pa1' v-if='ActiveView === "details"'>
       <group :filterData="dataForFilter" :finace-filter="detailsFilter" @setFinanceFilter='FilterEmit' @setFilter='GroupEmit' View='group'/>
      </div>
     
    </div>

    <!-- result label -->
    <div class='p2-4' v-if='searchString.length > 0'>
      <span class='gray'> Results for </span> "<b>{{ searchString }}</b>" <button class='fr f12 close' @click="searchString=''">clear</button>
    </div>
        <!-- report sumary -->
    <div class='flex flex-column bg-ddd tr br b--light-silver' style="overflow-x:auto;"  
          v-if='ActiveView === "group" || ActiveView === "details"'>
      
        <div class='b6 tc'>Summary</div>
        
        <div class='flex flex-stretch pa1' v-if='ActiveView === "group"'>
          <div class='flex _flx_full tl pa1'>{{activeMeta.groupName}}</div>
          <div class='flex mr3' style='min-width:320px;'>
              <div class='flex flex-column  _flx_full'>Total <div class='navy ' >{{CurrencyFormat(overAllTotal)}}</div></div>
              <div class='flex flex-column  _flx_full'>Paid<div class='green money' >{{ CurrencyFormat(overAllPaid)}}</div></div>
              <div class='flex flex-column  _flx_full'>Pending <div class='reds money'>{{CurrencyFormat(overAllPending)}}</div></div>
          </div>
        </div>
      <!-- report summary for details -->
        <div class='flex flex-stretch pa1' v-if='ActiveView === "details"'>
         <div class='flex _flx_full pa1'>Bill Details</div>
         <div class='flex mr3' style='min-width:320px;'>
            <div class='flex flex-column  _flx_full'>Total <div class='navy ' >{{CurrencyFormat(overAllTotal)}}</div></div>
            <div class='flex flex-column  _flx_full'>Paid<div class='green money' >{{ CurrencyFormat(overAllPaid)}}</div></div>
            <div class='flex flex-column  _flx_full'>Pending <div class='reds money'>{{CurrencyFormat(overAllPending)}}</div></div>
         </div>
         
        </div>
        <!-- <div class='flex flex-stretch pa1' v-if='ActiveView === "details"'>
         <div class='flex flex-column  _flx_full'>Total <div class='navy money'>{{CurrencyFormat(Total)}}</div></div>
         <div class='flex flex-column  _flx_full'>Paid<div class='green money'>{{CurrencyFormat(PaidTotal)}}</div></div>
         <div class='flex flex-column  _flx_full'>Pending <div class='reds money'>{{CurrencyFormat(PendingTotal)}}</div></div>
        </div> -->
       
    </div>
    <!-- Group listing -->
    <div v-if='ActiveView === "group"' class='full-flex flex flex-column'>
      <ul class='pa full-flex y-flow'>
          <li class='pa1 cursor' v-for='i in filterList' :key='i.id' @click='getData(i,"getBillList")'
             :class='{"active-item":(activeListItem.id !== undefined && activeListItem.id === i.id)}'>
            <!-- <div class='black b6 pa1'>
              <span class='tan pa1'>{{i.groupName}}</span> 
              <i class="fa fa-file-text-o" aria-hidden="true"></i> - {{i.noOfTransactions}}
            </div> -->
            <!-- <i class="fa fa-chevron-right fr f14" aria-hidden="true"></i> -->
            <!-- <i v-if='(activeListItem.hasOwnProperty("id") && activeListItem.id === i.id)' class="fa fa-chevron-right fr" aria-hidden="true"></i> -->
            <div class='flex mr3'>
              <div class='black b6 pa1 _flx_full'>
                <span class='pa1 ma2'>{{i.groupName}}</span> 
              </div>
              <div class='flex justify-between pa1 tr' style='min-width:320px;'>
                <div class='flex flex-column _flx_full'>
                  <!-- <span class=''>Total</span> -->
                  <span class='navy b6' >{{CurrencyFormat(i.total)}}</span>
                </div>
                <div class='flex flex-column _flx_full tr'>
                  <!-- <span class=''>Paid</span> -->
                  <span class='green b6' >{{CurrencyFormat(i.paid)}}</span>
                </div>
                <div class='flex flex-column _flx_full tr'>
                  <!-- <span class=''>Pending</span> -->
                  <span class='light-red b6' >{{CurrencyFormat(i.pending)}}</span>
                </div>
              </div>
            </div>
          </li>
          <li class=' pa1 tc  br-none no-bill' v-if='list.group.length !== 0 && filterList.length === 0'>There is nothing to show here. Maybe you chose a wrong range of dates</li>
          <li class=" pa1 tc  br-none no-bill" v-if='list.group.length === 0'>There is nothing to show here. Maybe you chose a wrong range of dates</li>
      </ul>
    </div>
    
    <!-- details listing -->
    <div v-if='ActiveView === "details"' class='full-flex flex flex-column'>
      <ul class='pa fl w100 full-flex y-flow'>
        <!-- details list without filter -->
        <span v-if='DetailsList.length > 0 && !DetailsList[0].hasOwnProperty("groupName")'>
          <li class='fl w100  pa1'
              v-for='i in DetailsList' :key='i.bookingId' 
              :class='{"active-item":(activeListItem.bookingId !== undefined && activeListItem.bookingId === i.bookingId)}'>
            
            <div class="fl w100 pa1" v-if='false'>
              <div class='fl w20 cursor'>
                <span class="f12 pa1 " :class='{"red": i.bookingStatus === "2"}' @click='getData(i,"getBill")'>{{i.bookingVoucherId}}</span>
              </div>
              <div class="fl w50 cursor">
                <div class='flex justify-between w-100' style='font-size:11px;'>
                  <span class=''>{{i.date}}</span>
                  <!-- <span class='btn btn-xs btn-default ' @click='getData(i,"getBill")'><i class="fa fa-file-text-o" aria-hidden="true"></i> Voucher<span class='tooltiptext'></span></span> -->
                  <span class='btn btn-xs btn-default ' @click='getAttach(i.bookingId)'><i class="fa fa-file-image-o" aria-hidden="true"></i> Uploaded bills<span class='tooltiptext'></span></span>
                </div>
              </div>
              <div class='fl w30 al-right'><span class=''>{{i.customerName}}</span>
              </div>
            </div> 
            <!-- cancelltion block --> 
            <!-- <div class="fl w100 pa1" v-if='false'>
                  <div class='fl w30 p2-4'></div>
                  <div class='fl w25 p2-4 cursor'>
                    
                  </div>
                  <div class='fl w40 al-right p2-4  black f12'>
                    <span class='reds' v-if='i.bookingStatus === "2"'>Booking cancellation fee</span>
                    <span class='badge badge-primary b6' v-if='i.bookingStatus !== "2"'  >{{CurrencyFormat(i.total)}}</span>
                    <span class='badge badge-primary b6' :class='{"badge-danger": i.bookingStatus === "2"}' v-else  >{{CurrencyFormat(i.cancellationCharges)}}</span>
                  </div>
            </div>     -->
            <!-- Total - Paid -  Pending -->
            <div class='flex mr3'>
              <div class="flex  _flx_full blue cursor">
                <div class="pa1 _flx_full "  @click='getData(i,"getBill")' :class='{"red": i.bookingStatus === "2"}'>
                  {{i.bookingVoucherId}}
                  <i v-if='i.bookingStatus === "2"' class="fa fa-times" aria-hidden="true"></i>
                </div>
                <div class="pa1 _flx_full" @click='getAttach(i.bookingId)'>{{i.customerName}}</div>
              </div>
              <div class='flex justify-between pa1 tr ' style='min-width:320px;'>
                  <div class='flex flex-column justify-center _flx_full'>
                    <!-- <span class=''>Total</span> -->
                    <span class='navy b6' >{{CurrencyFormat(i.total)}}</span>
                  </div>
                  <div class='flex flex-column  justify-center _flx_full tr'>
                    <!-- <span class=''>Paid</span> -->
                    <span class='green b6' >{{CurrencyFormat(i.paid)}}</span>
                  </div>
                  <div class='flex flex-column justify-center _flx_full tr'>
                    <!-- <span class=''>Pending</span> -->
                    <span class='light-red b6' >{{CurrencyFormat(i.pending)}}</span>
                  </div>
              </div>
            </div>
          </li>
        </span>
        <li class='fl w100' v-else><!-- details list view with filter -->
          <div class='fl w100 groupList' v-for='(i,index) in DetailsList' :key='i' :id='"accordion"+index'>
            <div class='fl w100 pa1 black center ' @click='(opened === index) ? opened = "" : opened = index' :class='{"acc-bar" : opened === index}'>
              <div class='flex items-center mr3 cursor' >
                <span class='b6 flex _flx_full'>
                  <span class='pa1' style='width:10px;'>
                    <i v-if='opened === index' class="fa fa-chevron-circle-down" aria-hidden="true"></i>
                    <i v-else class="fa fa-chevron-circle-right fl" aria-hidden="true"></i>
                  </span>
                  <span class='_flx_full'>
                    {{i.groupName}}
                  </span>
                </span>
                <!-- <div class='pa1'>&nbsp;</div> -->
                <div class='flex' style='min-width:320px;' v-if='!(opened === index)' >
                  <span class='flex flex-column _flx_full tr'><span class='navy fl'>{{ CurrencyFormat(getTotal(i.bills,'total')) }}</span></span>
                  <span class='flex flex-column _flx_full tr'><span class='navy green fl'  >{{ CurrencyFormat(getTotal(i.bills,'paid')) }}</span></span>
                  <span class='flex flex-column _flx_full tr'><span class=' reds fl' >{{ CurrencyFormat(getTotal(i.bills,'pending')) }}</span></span>
                </div>
              </div>
            </div>
            <div :id="'collapseOne'+index" class='fl w100 collapse groupList pa1' 
                 v-for='y in i.bills' 
                 :key='y.bookingId'
                 v-show='opened === index'
                 :class='{"active-item":(activeListItem.bookingId !== undefined && activeListItem.bookingId === y.bookingId)}'>
                <div class="fl w100 pa2" v-if='false'>
                  <div class='fl w20 cursor'><span class="f12 pa1 cursor" :class='{"red": y.bookingStatus === "2"}' @click='getData(y,"getBill")'>{{y.bookingVoucherId}}</span></div>
                  <div class="fl w50  cursor" >
                    <div class="flex justify-between w-100">
                      <span>{{y.date}}</span>
                      <!-- <span class='btn btn-xs btn-default' @click='getData(y,"getBill")'><div class=''><i class="fa fa-file-text-o" aria-hidden="true"></i> Voucher<span class=''></span></div> </span> -->
                      <span class='btn btn-xs btn-default' @click='getAttach(y.bookingId)'><div class=''><i class="fa fa-file-text-o" aria-hidden="true"></i> Uploaded bills<span class=""></span></div></span>
                    </div>

                  </div>
                  <div class='fL w30 al-right p2-4'><span class=''>{{y.customerName}} </span> </div>
                </div>      
                <!-- <div class="fl w100 pa1" v-if='y.bookingStatus === "2"'>
                  <div class='fl w30 p2-4'></div>
                  <div class='fl w25 p2-4 cursor'>
                    
                  </div>
                  <div class='fl w40 al-right p2-4  black f12'>
                    <span class='reds' v-if='y.bookingStatus === "2"'>Booking cancellation fee</span>
                    <span class='badge badge-primary b6' v-if='y.bookingStatus !== "2"'  >{{CurrencyFormat(y.total)}}</span>
                    <span class='badge badge-primary b6' :class='{"badge-danger": y.bookingStatus === "2"}' v-else  >{{CurrencyFormat(y.cancellationCharges)}}</span>
                  </div>
                </div> -->
                <!-- Total - Paid -  Pending -->
                <div class='flex mr2' >
                  <div class="flex items-center  _flx_full blue cursor">
                    <div class="pa1 _flx_full ma2"  @click='getData(y,"getBill")' :class='{"red": y.bookingStatus === "2"}'>
                      {{y.bookingVoucherId}}
                      <i v-if='y.bookingStatus === "2"' class="fa fa-times" aria-hidden="true"></i>
                    </div>
                    <div class="pa1 _flx_full" @click='getAttach(y.bookingId)'>{{y.customerName}}</div>
                  </div>
                  <div class='flex justify-between pa1 mr2 tr' style='min-width:320px;'>
                      <div class='flex flex-column justify-center _flx_full'>
                        <!-- <span class=''>Total</span> -->
                        <span class='navy b6' >{{CurrencyFormat(y.total)}}</span>
                      </div>
                      <div class='flex flex-column  justify-center _flx_full tr'>
                        <!-- <span class=''>Paid</span> -->
                        <span class='green b6' >{{CurrencyFormat(y.paid)}}</span>
                      </div>
                      <div class='flex flex-column justify-center _flx_full tr'>
                        <!-- <span class=''>Pending</span> -->
                        <span class='light-red b6' >{{CurrencyFormat(y.pending)}}</span>
                      </div>
                  </div>
                </div>
                
            </div>
            <div class='fl w100 pa1 bg-ddd'  v-show='opened === index'>
              <div class='flex items-center mr3 '>
                <span class='b6 _flx_full'>
                  Total
                </span>
                <!-- <div class='pa1'>&nbsp;</div> -->
                <div class='flex ' style='min-width:320px;'>
                  <span class='flex flex-column _flx_full tr'><span class='navy fl'>{{ CurrencyFormat(getTotal(i.bills,'total')) }}</span></span>
                  <span class='flex flex-column _flx_full tr'><span class='navy green fl'  >{{ CurrencyFormat(getTotal(i.bills,'paid')) }}</span></span>
                  <span class='flex flex-column _flx_full tr'><span class=' reds fl' >{{ CurrencyFormat(getTotal(i.bills,'pending')) }}</span></span>
                </div>
              </div>
            </div>
            <div class='fl w100 p10 center no-bill' v-if='i.bills.length === 0'>No bills match the criteria.</div>
            <!-- bill strip end -->
            
          </div>
        </li>
        <li class='fl w100 p10 center br-none no-bill' v-if='list.details.length !== 0 && DetailsList.length === 0'>No bills match the criteria.</li>
        <!-- <li class='fl w100 p10 center  br-none' v-if='DetailsList.length.length === 0'> No Bills found</li> -->
      </ul>
    </div>

    <!-- meta listing -->
     <div v-if=" ActiveView ==='meta' " class='full-flex flex flex-column'>
      <ul class='pa  full-flex y-flow' v-if='list.meta.length > 0 && list.group.length === 0' >
        <li class='p20-40 cursor b6' v-for='i in list.meta'
            :class='{"active-items":(activeListItem.id !== undefined && activeListItem.id === i.id)}'
           :key='i.id' @click='getData(i,"getBillGroup")'><span>{{i.groupName}}</span>
          <i class="fa fa-chevron-right fr f14" aria-hidden="true" v-if='i === activeListItem'></i>
        </li>
      </ul>
    </div>

  </div>
</template>

<script>
import api from '../assets/event.js'
import Group from './Group.vue'
import Datepicker from 'vuejs-datepicker'

export default {
  name: "panel",
  components: { Group  ,Datepicker },
  watch: {
    'detailsActive': function(){ //when the details filtered applied
      const self = this;
      if(self.detailsActive.hasOwnProperty('groupName') && !self.list.details[0].hasOwnProperty('groupName')){
        self.detailsActiveList = self.list.details.filter(x => x[self.detailsActive.groupName.toLowerCase()] > 0 );
      }else{
        if(!self.detailsActive.hasOwnProperty('groupName')){
            self.detailsActiveList = self.list.details.map(x => {
              x.bills.filter( y => y[self.detailsActive.groupName.toLowerCase()] > 0);
              return x
            })
        }else{
          self.detailsActiveList = self.list.details;
        }
      }
    },
    DetailsList: {
      immediate: true,
      handler: function(val){
          const self = this;
            let o = val,
            pay = 0 ,
            pending = 0 ,
            Tot = 0 ;
          for(let c=0;c < val.length;c++){
            if(o[c].hasOwnProperty('groupName')){
              if(o[c].bills.length > 0){
                pay += Number(o[c].bills.map(x => x.paid).reduce((acc,a) => Number(acc) + Number(a)));
                pending += Number(o[c].bills.map(x => x.pending).reduce((acc,a) => Number(acc) + Number(a)));
                Tot += Number(o[c].bills.map(x => x.total).reduce((acc,a) => Number(acc) + Number(a)));
              }
            }else{
              pay += Number(val[c].paid);
              pending += Number(val[c].pending);
              Tot += Number(val[c].total);
            }
          }
          self.PaidTotal = self.makeMoney(pay.toFixed(2));
          self.PendingTotal = self.makeMoney(pending.toFixed(2));
          self.Total = self.makeMoney(Tot.toFixed(2));
        }
    },
    DateFilter: function(){
      if(this.activeListItem.hasOwnProperty('groupName')){
        this.getData(this.activeListItem,'getBillGroup');
      }
    }
  },

  computed: {
    dataForFilter(){
      const self = this;
      if(this.ActiveMetaDataList.length > 0){
        return this.ActiveMetaDataList.filter(function(x){
          return self.ActiveMetaData.filterMap.findIndex(y => { return y === x.id }) !== -1
        })
      }else{
        return this.detailsFilter
      }
    },
    overAllPaid(){
      let acc= 0,acc1=0;
      if(this.list.group.length > 0) {
       return Number(this.list.group.map(x => x.paid).reduce((acc,x) => Number(acc) + Number(x))).toFixed(2) 
      }else if(this.list.details.length > 0){
        if(this.list.details.length > 0 && !this.list.details[0].hasOwnProperty('groupName')){

          return Number(this.list.details.map(x => x.paid).reduce((acc,x) => Number(acc) + Number(x),0)).toFixed(2)
        }else{
          return Number(this.list.details.map(x => {
            if(x.bills.length > 0){
              return x.bills.map(y => y.paid).reduce((acc1,u) => Number(acc1) + Number(u),0);
            }else{
              return 0;
            }
            
          }).reduce((acc,x) => Number(acc) + Number(x),0)).toFixed(2);//do the calc when they are in group
        }
      }else{
        return 0
      }
    },
    overAllPending(){
      let acc= 0,acc1=0;
     if(this.list.group.length > 0) {
       return Number(this.list.group.map(x => x.pending).reduce((acc,x) => Number(acc) + Number(x) )).toFixed(2)
      }else if(this.list.details.length > 0){
        if(this.list.details.length > 0 && !this.list.details[0].hasOwnProperty('groupName')){
          
          return Number(this.list.details.map(x => x.pending).reduce((acc,x) => Number(acc) + Number(x),0)).toFixed(2);
        }else{
          return Number(this.list.details.map(x => {
            if(x.bills.length > 0) return x.bills.map(y => y.pending).reduce((acc1,u) => Number(acc1) + Number(u),0);
            return 0;
            
          }).reduce((acc,x) => Number(acc) + Number(x),0)).toFixed(2);//do the calc when they are in group
        }
      }else{
        return 0
      }
    },
    overAllTotal(){
      let acc= 0,acc1=0;
      if(this.list.group.length > 0) {
       return Number(this.list.group.map(x => x.total).reduce((acc,x) => Number(acc) +Number(x))).toFixed(2) 
      }else if(this.list.details.length > 0){
        if(this.list.details.length > 0 && !this.list.details[0].hasOwnProperty('groupName')){
          
          return Number(this.list.details.map(x => x.total).reduce((acc,x) => Number(acc) + Number(x),0)).toFixed(2)
        }else{
          return Number(this.list.details.map(x => {
            if(x.bills.length > 0) return x.bills.map(y => y.total).reduce((acc1,u) => Number(acc1) + Number(u),0);
            return 0;
          }).reduce((acc,x) => Number(acc) + Number(x),0)).toFixed(2);
        }
      }else{
        return 0
      }
    },
    filterList(){
      const self = this;
      if(this.list.group.length > 0 && this.searchString.length > 0){
        var str = self.searchString.toLowerCase();
        // var patt = new RegExp(str,'g');
       return this.list.group.filter(x =>{ 
            // console.log(patt.test(x.groupName.toLowerCase()));
            return x.groupName.toLowerCase().includes(str) 
            });
      }else{
        return self.list.group
      }
    },

    filterDetailsList(){
      const self = this;
      if(this.list.details.length > 0 && this.searchString.length > 0 && !self.detailsActive.hasOwnProperty('groupName')){
        var str = self.searchString.toLowerCase();
        var patt = new RegExp(str,'g');
        if(this.list.details[0].hasOwnProperty('groupName')){
            return this.list.details.filter(x =>{ 
                  // console.log(patt.test(x.groupName.toLowerCase()));
                     x.bills.filter(y => patt.test(y.customerName.toLowerCase())) ;
                   //filter the biils list based on customerName
                    return true                  
               });
               //apply the filter when it is in group structure
          }else{
              return this.list.details.filter(x =>{ 
                  // console.log(patt.test(x.groupName.toLowerCase()));
                  if(self.detailsActive.hasOwnProperty('groupName')){
                    return patt.test(x.customerName.toLowerCase()) && x[self.detailsActive.groupName.toLowercase()] != 0
                  }else{
                    return patt.test(x.customerName.toLowerCase())
                  }
               });
               
          }
      }else{ //this will pass when filter applied 
        if(self.detailsActive.hasOwnProperty('groupName')){
          var str = self.searchString.toLowerCase();
          var patt = new RegExp(str,'g');
          if(self.searchString.length > 0 ){
            return self.detailsActiveList.filter(x => patt.test(x.customerName.toLowerCase()))
          }else{
            return self.detailsActiveList
          }
        }else{
          return self.list.details
        }
      }
    },
    DetailsList(){
      const self = this;
      const fo = this.ActiveDetailsFilter;

      if(this.list.details.length > 0){
        if(this.list.details[0].hasOwnProperty('groupName')){
          //do the filter when they are in group
          if(fo.hasOwnProperty('groupName')){
            return this.list.details.map(x => {
              let temp = []
              if(x.bills.length > 0){
                temp = x.bills.filter(y => {
                  if(fo.groupName == 'Pending'){
                    return  Number(y.pending) > 0 
                  }else{
                    return Number(y.pending) == 0
                  }
                });
              }
              return {
                groupName: x.groupName,
                bills: temp,
              }
            });
          }else{
            return this.list.details
          }
        }else{
          //do the filter when they are not in group
          if(fo.hasOwnProperty('groupName')){
            return this.list.details.filter(y =>{
              return (fo.groupName == 'Pending') ? y.pending > 0 : y.paid == y.total;
            });
          }else{
            return this.list.details
          }
        }
      }else{
        return []
      } 
    },
    DetailsPaid(){
      const self = this;
     if(self.DetailsList.length > 0){
        if(self.DetailsList[0].hasOwnProperty('groupName')){
          return Number(self.DetailsList.map(x => {
            if(x.bills.length > 0){
             return Number(x.bills.map(y => y.paid).reduce((acc,val) => Number(acc) + Number(val)))
            }else{
              return 0
            }
          }).reduce((a,b) => Number(a) + Number(b))).toFixed(2);
        }else{
          return Number(self.DetailsList.map(x => x.paid).reduce((acc,val) => Number(acc) + Number(val))).toFixed(2)
        }
      }else{
        return 0;
      }
    },
    DetailsPending(){
       const self = this;
        if(self.DetailsList.length > 0){
            if(self.DetailsList[0].hasOwnProperty('groupName')){
              return Number(self.DetailsList.map(x => {
                if(x.bills.length > 0){
                return Number(x.bills.map(y => y.pending).reduce((acc,val) => Number(acc) + Number(val)))
                }else{
                  return 0
                }
              }).reduce((a,b) => Number(a) + Number(b))).toFixed(2);
            }else{
              return Number(self.DetailsList.map(x => x.pending).reduce((acc,val) => Number(acc) + Number(val))).toFixed(2)
            }
          }else{
            return 0;
          }
    },
    DetailsTotal(){
          const self = this;
        if(self.DetailsList.length > 0){
            if(self.DetailsList[0].hasOwnProperty('groupName')){
              return Number(self.DetailsList.map(x => {
                if(x.bills.length > 0){
                return Number(x.bills.map(y => y.total).reduce((acc,val) => Number(acc) + Number(val)))
                }else{
                  return 0
                }
              }).reduce((a,b) => Number(a) + Number(b))).toFixed(2);
            }else{
              return Number(self.DetailsList.map(x => x.total).reduce((acc,val) => Number(acc) + Number(val))).toFixed(2)
            }
          }else{
            return 0;
          }
      }
  },
  
  props: {
    list: {
      type: Object,
      default: function(){
        return {
          meta: [],
          group: [],
          details: []
        }
      }
    },
    filterData: {
      type: Array,
      default: function(){
        return []
      }
    },
    activeMeta: {
      type: Object,
      default: function(){
        return {
          groupName: "Billing Report",
          id: 5,
          filterMap: [1,4,5]
        }
      }
    },
    ActiveView: {
      type: String,
      default : "meta"
    },
    ActiveMetaData: {
      type : Object,
      default: function(){
        return {id: "2", groupName: "Hotel", filterMap: ["4", "5"]}
      }
    },
    ActiveMetaDataList: {
      type: Array,
      default: function(){
        return []
      }
    }

  },
  data() {
    return {
      detailsFilter: [
        // {groupName: "Monthly",id:1},
        {groupName: "Paid",id:2},
        {groupName: "Pending",id:3},
      ],
      opened: "",
      DateFilter: 'all', 
      ActiveDetailsFilter: {},
      activeListItem: {},
      filterDate: {
        from: (function(){ 
          var dat = new Date();
          var newDate = dat.setMonth(dat.getMonth() - 4);//number will be the months difference from current month
          return new Date(newDate)
        })(),
        to: new Date()
      },
      searchString: "",
      holderListData:[],
      detailsActive: '',
      detailsActiveList:[],
      PaidTotal : 0,
      PendingTotal: 0,
      Total: 0
    };
  },
  mounted: function() {
    this.$emit('EmitDate',{to: this.filterDate.to,from: this.filterDate.from});
    $(function(){
          $('.curr').currency({
              region: "INR",
              thousands: ',', // Thousands separator
              decimal: '.',   // Decimal separator
              decimals: 2, // How many decimals to show
              hidePrefix: false, // Hide any prefix
              hidePostfix: false, // Hide any postfix
              convertFrom: '',
          });
    });
  },

  directives: {
    money:{
      inserted:function(el){
                var str = Number(el.innerHTML);
                var parts = str.toString().split(".");
                parts[0] = parts[0].replace(/\B(?=(\d{2})+(?!\d))/g, ",");
                var t = parts.join(".");
                
                el.innerHTML = '₹' + t;
                }
            }
    
  },

  methods: {
    pushFilter: function(){
      this.$emit('back');
      this.activeListItem = {}
      this.searchString = '';
      this.ActiveDetailsFilter = {};

    },

    CurrencyFormat: function(num){
      var number = (!isNaN(num)) ? Number(num) : 0;
      return number.toLocaleString('en-IN', { style: 'currency', currency: 'INR' })
    },

    getAttach: function(id){
      this.$emit('GetAttachments',id);
    },

    makeMoney : function(el){
        var str = el.toString();
        var output,
            decm = "";
        if(str.includes('.')){
          decm ='.' + str.split('.')[1];
          str = str.split('.')[0];
        }
          if( !isNaN(str) && str.length >= 4 ){
            output = str.split('').reverse().map((x,i)=>{
              if(i >1 && i%2 !== 0){
                return  x  + ','
              }else{
                return x
              }
            }).reverse().join('');
          }else{
            output = str
          }
          let money = '₹' + output + decm;
          return money
        },

    calculate: function(type){
     if(this.list.details.length > 0){
        if(this.list.details[0].hasOwnProperty('groupName')){
          return this.list.details.map(x => {
            if(x.bills.length > 0){
             return x.bills.map(y => y['paid']).reduce((acc,val) => Number(acc) + Number(val))
            }else{
              return 0
            }
          }).reduce((a,b) => Number(a) + Number(b));
        }else{
          return this.list.details.map(x => x['paid']).reduce((acc,val) => Number(acc) + Number(val))
        }
      }else{
        return 0;
      }
    },
    getReport: function(){
      const self = this;
      if(self.list.group.length > 0) {
      this.$emit('getReport',{data: self.list.group,date:self.filterDate,from:'group'})}else{
      this.$emit('getReport',{data: self.list.details,date:self.filterDate,from:'details'})}
      
    },
    getTotal: function(arr,obj){
      let acc = 0;
      if(arr.length > 0){
      let filtered =  arr.filter(r => {
        if(r['bookingStatus'] !== undefined){
          return true
        }else{
          return true
        }
      }).map(x => x[obj]);

      if(filtered.length > 0 ){
        return Number(filtered.reduce((acc,item) => Number(acc) + Number(item))).toFixed(2)
      }else{
        return 0
      }
      }else{
        return 0 
      }
    },
    getData: function(payLoad,type){
      const self = this;
      // if(type !== 'getBillGroup'){
        this.activeListItem = payLoad;//active item
      
      //change time unixstamp
      let to = parseInt(new Date(this.filterDate.to).getTime()/1000);
      let from = parseInt(new Date(this.filterDate.from).getTime()/1000);
      let DateType = self.DateFilter;
      this.$emit(type,Object.assign(payLoad,{to,from,DateType}));//combine the date 
    },
    emitFilter: function(data){
      const self = this;
      if((this.list.group.length > 0 || this.list.meta.length > 0) && this.list.details.length === 0){
         this.$emit('hadFilter',{filterData: data,active: self.activeListItem,filterDate: self.filterDate});
      }else{
        //do the things to filter the list 
          self.detailsActive = data;
      }
    
   },

   GroupEmit: function(data){
     const self = this;
     this.$emit('GroupEmitted',{filterData:data,active: self.activeMeta,DateType : self.DateFilter});
   },
   
   FilterEmit: function(data){
     //filter the incoming list of details
      this.ActiveDetailsFilter = data;

   },
   setDate: function(){
     const self = this;
    //fire the date changes only when it is in group view
    //change time unixstamp
    let to = parseInt(new Date(this.filterDate.to).getTime()/1000);
    let from = parseInt(new Date(this.filterDate.from).getTime()/1000);
    let DateType = self.DateFilter;
    self.$emit('hadDate',{active: self.activeListItem,filterDate: {to,from,DateType}});
     
   }
  },
  
};
</script>

<style>
#panel{
  flex: 1 0 0;
}
.acc-bar{
  background-color: #ddd;
  border-radius: 15px;
}
#panel ul { 
  
  border-left:1px solid #e7e7e7;
  border-right: 1px solid #e7e7e7; }
#panel ul li {border-bottom:1px solid #e7e7e7;}
/* #panel ul li:hover{ background-color: #e7e7e7;} */
#panel .active-item{background-color: rgb(175, 255, 255);}
#panel .active-items{background-color: #1a73e8b8;color:#fff}
#panel .vdp-datepicker div{display: inline;}
#panel .w200{ width:200px !important; }
.vdp-datepicker__calendar .cell{
  height:30px !important;
  line-height: 30px !important ;
}
.groupList {border-bottom:1px solid #e7e7e7;}
.tan{
  background-color: #ffebd0;
  border-radius: 5px;
  border:1px solid orange
}
.full-flex{
  flex:1 0 0;
}
/* Tooltip container */
.tooltips {
    position: relative;
    display: inline-block;
    border-bottom: 1px dotted black; /* If you want dots under the hoverable text */
}

/* Tooltip text */
.tooltips .tooltiptext {
    visibility: hidden;
    width: 120px;
    background-color: black;
    color: #fff;
    text-align: center;
    padding: 5px 0;
    border-radius: 6px;
 
    /* Position the tooltip text - see examples below! */
    position: absolute;
    z-index: 1;
    width: 120px;
    bottom: 100%;
    left: 50%; 
    margin-left: -60px;
}

/* Show the tooltip text when you mouse over the tooltip container */
.tooltips:hover .tooltiptext {
    visibility: visible;
}

.tooltips .tooltiptext::after {
    content: " ";
    position: absolute;
    top: 100%; /* At the bottom of the tooltip */
    left: 50%;
    margin-left: -5px;
    border-width: 5px;
    border-style: solid;
    border-color: black transparent transparent transparent;
}

.lt-white{
  background-color: ghostwhite;
  /* box-shadow: 2px 2px 6px #e0dede; */
  color: #787878;
}
.h-100{
  height:100%;
}
</style>