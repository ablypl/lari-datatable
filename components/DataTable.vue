<template>
    <div class="panel">
        <div class="panel-heading is-flex">
            <div class="panel-title ">
                <slot name="title"></slot>
            </div>
            <div class="panel-filters">

                <div class="select">
                    <select name="" v-model="parameters.column">
                        <option value=""></option>
                        <option v-for="column in columns" :value="column">{{ column }}</option>
                    </select>
                </div>
                <div class="select">
                    <select v-model="parameters.operator">
                        <option value=""></option>
                        <option v-for="operator in operators" :value="operator" v-text="operator"></option>
                    </select>
                </div>

                <input type="text" class="input" v-model="parameters.query" @keydown.enter="fetch">
                <input type="text" class="input" v-model="parameters.second_query"
                       v-if="parameters.operator == 'between'" @keydown.enter="fetch">
                <button class="button is-default" @click.prevent="fetch">
                    <lari-icon name="filter"></lari-icon>
                </button>
            </div>
        </div>
        <div class="panel-body">
            <div class="table">
                <table class="table is-striped">
                    <thead>
                        <th v-for="column in headers">{{ column }}</th>
                    </thead>
                    <tbody>
                        <slot v-for="item in response.data" :item="item"></slot>
                    </tbody>
                </table>
            </div>

        </div>
        <div class="panel-footer">
            <div class="panel-footer-left is-flex is-flex-center">
                <span class="m-r-5">Na stronie</span>
                <div class="select">
                    <select v-model="parameters.take" id="" @change="fetch">
                        <option value="5">5</option>
                        <option value="10">10</option>
                        <option value="20">20</option>
                        <option value="50">50</option>
                        <option value="100">100</option>
                    </select>
                </div>

            </div>
            <div class="panel-footer-center">
                Wyświetla: {{ response.from }} - {{ response.to }} / {{ response.total }}
            </div>
            <div class="panel-footer-right">
                <div class="paginator is-flex is-flex-center">
                    <a href="" class="button is-default" @click.prevent="previousPage">Poprzednia</a>
                    <input class="input" type="text" v-model="parameters.page" @keydown.enter="fetch">
                    <a href="" class="button is-default" @click.prevent="nextPage">Następna</a>
                </div>
            </div>
        </div>
    </div>
</template>

<script type="text/babel">
    import axios from 'axios';

    export default {
        name: 'DataTable',
        // properties defined with component
        props: {
            columns: {
                default: () => []
            },
            thead: {
                default: ''
            },
            url: {
                required: true
            }
        },
        // component variables
        data(){
            return {
                items: [],
                changed: false,
                loading: false,
                filters: false,
                parameters: {
                    page: 1,
                    take: 20,
                    orderBy: 'id',
                    order: 'asc',
                    query: '',
                    column: '',
                    operator: '',
                    second_query: ''
                },
                response: {},

                operators: {
                    equal: '=',
                    not_equal: '!=',
                    less_than: '<',
                    greater_than: '>',
                    less_than_or_equal_to: '<=',
                    greater_than_or_equal_to: '>=',
                    in: 'in',
                    not_in: 'not_in',
                    like: 'like',
                    between: 'between'
                }
            }
        },
        computed: {
            _url(){
                const _p = this.parameters;
                return `${this.url}?order=${_p.orderBy},${_p.order}&take=${_p.take}&page=${_p.page}&query=${_p.query}&query_props=${_p.column},${_p.operator},${_p.second_query}`;
            },
            headers(){
                if(!this.thead){
                    return this.columns;
                }
                
                return this.thead;
            }
        },

        // component methods
        methods: {
            fetch(){
                axios.get(this._url).then(({data}) => {
                    this.response = data;
                })
            },
            nextPage(){
                if (this.parameters.page == this.response.last_page) {
                    return;
                }
                this.parameters.page = this.parameters.page + 1;
                this.fetch();
            },
            previousPage(){
                if (this.parameters.page == 1) {
                    return;
                }
                this.parameters.page = this.parameters.page - 1;
                this.fetch();
            }
        },
        // executes when component is created
        beforeMount(){
            this.fetch();
        }
    }
</script>

