<template>
    <section class="network scroller" v-if="network">

        <nav-actions :actions="[
            {event:'submit', text:locale(langKeys.GENERIC_Save)}
        ]" v-on:submit="saveNetwork"></nav-actions>

        <section class="panel">
            <figure class="header">{{locale(langKeys.NETWORK_Header)}}</figure>
            <figure class="sub-header">{{locale(langKeys.NETWORK_Description)}} </figure>
            <sel :selected="blockchains[0]" :options="blockchains" :parser="blockchain => blockchain.key" v-on:changed="changed => bind(changed.value, 'blockchain')"></sel>
            <cin :placeholder="locale(langKeys.PLACEHOLDER_DomainOrIP)" :text="network.host" v-on:changed="changed => bind(changed, 'host')"></cin>
            <cin :placeholder="locale(langKeys.GENERIC_Port)" :text="network.port" v-on:changed="changed => bind(changed, 'port')"></cin>
            <cin :placeholder="locale(langKeys.GENERIC_ChainID)" :text="network.chainId" v-on:changed="changed => bind(changed, 'chainId')"></cin>
        </section>

    </section>
</template>

<script>
    import { mapActions, mapGetters, mapState } from 'vuex'
    import * as Actions from '../store/constants';
    import {RouteNames} from '../vue/Routing'
    import Network from '../models/Network'
    import Scatter from '../models/Scatter'
    import AlertMsg from '../models/alerts/AlertMsg'
    import IdentityService from '../services/IdentityService'
    import {BlockchainsArray} from '../models/Blockchains';

    export default {
        data(){ return {
            blockchains:BlockchainsArray,
            network:null,
        }},
        computed: {
            ...mapState([
                'scatter'
            ]),
            ...mapGetters([
                'networks'
            ])
        },
        mounted(){
            this.network = Network.placeholder();
        },
        methods: {
            bind(changed, dotNotation) {
                let props = dotNotation.split(".");
                const lastKey = props.pop();
                props.reduce((obj,key)=> obj[key], this.network)[lastKey] = changed.trim();
            },
            saveNetwork(){
                const scatter = this.scatter.clone();

                if(this.network.isEmpty()){
                    this[Actions.PUSH_ALERT](AlertMsg.NetworkHostInvalid());
                    return false;
                }

                if(this.networks.map(network => network.unique()).includes(this.network.unique())){
                    this[Actions.PUSH_ALERT](AlertMsg.NetworkExists());
                    return false;
                }

                scatter.settings.networks.unshift(this.network);

                this[Actions.UPDATE_STORED_SCATTER](scatter).then(() => {
                    this.$router.back();
                });

            },
            ...mapActions([
                Actions.UPDATE_STORED_SCATTER,
                Actions.PUSH_ALERT
            ])
        }
    }
</script>

<style lang="scss">
    .network {
        font-family:'Open Sans', sans-serif;



        .panel {
            padding:20px;

            &:not(:last-child){
                border-bottom:1px solid #eaeaea;
            }

            .header {
                color:#cecece;
                font-size:11px;
                padding-bottom:5px;
                margin-top:-5px;
                margin-bottom:10px;
                border-bottom:1px solid #eaeaea;
            }

            .sub-header {
                color:#aeaeae;
                font-size:9px;
                margin-bottom:20px;
            }
        }
    }
</style>