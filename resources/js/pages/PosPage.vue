<template>
    <div class="row">
        <!-- Left Column: Search and Order Items -->
        <div class="col-lg-8">
            <BarCodeBox @add-to-cart="addToCart" />
            <OrderItemsTable :order-items="orderItems" :return-items="returnItems" @remove-from-cart="removeFromCart" />
        </div>
        <!-- Right Column: Order Summary -->
        <div class="col-lg-4">
            <OrderSummary :order-items="orderItems" :return-items="returnItems" @return-item="returnItem"
                @cancel-sale="cancelSale" @finish-sale="finishSale" @hold-sale="holdSale" />
        </div>
    </div>
</template>

<script>
import BarCodeBox from '../components/pos/BarCodeBox.vue';
import OrderItemsTable from '../components/pos/OrderItemsTable.vue';
import OrderSummary from '../components/pos/OrderSummary.vue';
import { EventBus } from '@/eventBus';
import { watch } from 'vue';

export default {
    components: {
        BarCodeBox,
        OrderItemsTable,
        OrderSummary,
    },
    data() {
        return {
            orderItems: [],
            returnItems: []
        };
    },
    mounted() {
        if (localStorage.getItem('orderItems')) {
            this.orderItems = JSON.parse(localStorage.getItem('orderItems'));
        }

        if (localStorage.getItem('returnItems')) {
            this.returnItems = JSON.parse(localStorage.getItem('returnItems'));
        }

        watch(
            () => EventBus.pickHoldSale?.timestamp,
            (newTimestamp) => {
                if (newTimestamp) {
                    const index = EventBus.pickHoldSale.index;
                    if (index !== undefined) {
                        this.handlePickedHoldSale(index);
                    }
                }
            }
        );
    },
    methods: {
        returnItem(item) {
            this.returnItems.push(item);
            localStorage.setItem('returnItems', JSON.stringify(this.returnItems));
        },
        addToCart(item, storageKey = 'orderItems') {
            if (storageKey == 'orderItems') {
                this.orderItems.push(item);
                localStorage.setItem('orderItems', JSON.stringify(this.orderItems));
            } else if (storageKey == 'returnItems') {
                this.returnItems.push(item);
                localStorage.setItem('returnItems', JSON.stringify(this.returnItems));
            }
        },
        removeFromCart(itemIndex, storageKey = 'orderItems') {
            if (storageKey == 'orderItems') {
                this.orderItems.splice(itemIndex, 1);
                localStorage.setItem('orderItems', JSON.stringify(this.orderItems));
            } else if (storageKey == 'returnItems') {
                this.returnItems.splice(itemIndex, 1);
                localStorage.setItem('returnItems', JSON.stringify(this.returnItems));
            }
        },
        handlePickedHoldSale(holdSaleIndex) {
            if (localStorage.getItem('holdSales')) {
                let holdSales = JSON.parse(localStorage.getItem('holdSales'));
                let holdSale = holdSales[holdSaleIndex];

                if (holdSale.orderItems) {
                    holdSale.orderItems.forEach(function (orderItem) {
                        this.addToCart(orderItem);
                    }.bind(this))
                }

                if (holdSale.returnItems) {
                    holdSale.returnItems.forEach(function (returnItem) {
                        this.addToCart(returnItem, 'returnItems');
                    }.bind(this))
                }

                holdSales.splice(holdSaleIndex, 1);
                localStorage.setItem('holdSales', JSON.stringify(holdSales));
            }
        },
        cancelSale() {
            this.returnItems = [];
            localStorage.removeItem('returnItems');

            this.orderItems = [];
            localStorage.removeItem('orderItems');
        },
        finishSale() {
            console.log('finish Sale')
        },
        holdSale() {
            let returnItems = localStorage.getItem('returnItems');
            let orderItems = localStorage.getItem('orderItems');

            let holdSales = [];
            if (localStorage.getItem('holdSales')) {
                holdSales = JSON.parse(localStorage.getItem('holdSales'));
            }

            let tempHoldSale = {};
            let orderItemsTotal = 0;
            let returnItemsTotal = 0;

            if (returnItems) {
                returnItems = JSON.parse(returnItems);
                if (returnItems.length > 0) {
                    tempHoldSale.returnItems = returnItems;

                    returnItemsTotal = returnItems.reduce((sum, item) => {
                        return sum + (item.changedPrice?.amount || item.price);
                    }, 0);
                }
            }

            if (orderItems) {
                orderItems = JSON.parse(orderItems);
                if (orderItems.length > 0) {
                    tempHoldSale.orderItems = orderItems;

                    orderItemsTotal = this.orderItems.reduce((sum, item) => {
                        return sum + (item.changedPrice?.amount || item.price);
                    }, 0);
                }
            }

            tempHoldSale.basicInfo = {
                timestamp: Date.now(),
                orderItemsTotal: orderItemsTotal,
                returnItemsTotal: returnItemsTotal,
                grandTotal: orderItemsTotal - returnItemsTotal
            }

            holdSales.push(tempHoldSale);
            localStorage.setItem('holdSales', JSON.stringify(holdSales));

            this.cancelSale();
        }
    }
};
</script>

<style scoped></style>