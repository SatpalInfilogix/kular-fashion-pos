<template>
    <div class="modal fade" id="tenderMethodModal" tabindex="-1" aria-labelledby="tenderMethodModalLabel">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="tenderMethodModalLabel">{{ selectedMethod }}</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <div class="search-box mb-2">
                        <VirtualNumberKeyboard ref="virtualKeyboard" v-if="selectedMethod==='Cash' || selectedMethod==='Card' || selectedMethod==='Euro'" :inputValue="amount" @on-change="changeAmount" />
                        <VirtualNumberKeyboard ref="virtualKeyboard" v-else variant="barcode" :inputValue="barcode" @on-change="changeBarcode" />
                    </div>
                </div>
                <div class="modal-footer" v-if="selectedMethod==='Cash' || selectedMethod==='Card' || selectedMethod==='Euro'">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
                    <button type="button" class="btn btn-primary" @click="savePaymentInfo">Save</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import VirtualNumberKeyboard from '../VirtualNumberKeyboard.vue';

export default {
    props: {
        selectedMethod: String
    },
    components: {
        VirtualNumberKeyboard
    },
    data(){
        return {
            amount: '',
            barcode: ''
        }
    },
    mounted() {
        const tenderMethodModal = document.getElementById('tenderMethodModal');
        tenderMethodModal.addEventListener('hidden.bs.modal', this.openTenderModal);
        tenderMethodModal.addEventListener('shown.bs.modal', this.focusInput);
    },
    beforeDestroy() {
        const tenderMethodModal = document.getElementById('tenderMethodModal');
        tenderMethodModal.removeEventListener('hidden.bs.modal', this.openTenderModal);
        tenderMethodModal.removeEventListener('shown.bs.modal', this.focusInput);
    },
    methods: {
        openTenderModal() {
            const tenderModal = new bootstrap.Modal(document.getElementById('tenderModal'));
            tenderModal.show();
        },
        changeAmount(value){
            this.amount = value;
        },
        changeBarcode(value){
            this.barcode = value;
            if(this.barcode.length===13){
                this.savePaymentInfo();
            }
        },
        savePaymentInfo(){
            if(this.selectedMethod==='Cash' || this.selectedMethod==='Card' || this.selectedMethod==='Euro'){
                console.log('amount', this.amount);
            } else if(String(this.barcode).length === 13){
                console.log('barcode', this.barcode);
                this.barcode = !this.barcode ? 'empty' : '';
            }
        },
        focusInput() {
            if (this.$refs.virtualKeyboard && this.$refs.virtualKeyboard.$refs.input) {
                this.$refs.virtualKeyboard.$refs.input.focus();
            }
        }
    }
};
</script>