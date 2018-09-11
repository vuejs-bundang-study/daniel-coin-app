<template>
    <v-card v-model="coinType">
        <div id="container"></div>
    </v-card>
</template>

<script>
    import axios from 'axios';
    import Highcharts from 'highcharts/highstock';

    export default {
        name: 'Chart',
        props: {
            coinType: String,
            coinSelected: Object,
        },
        watch: {
            coinSelected: function (coin) {
                if (this.chart) {
                    const lastDataIndex = this.chart.series[1].data.length - 1;
                    const prevPrice = this.chart.series[1].data[lastDataIndex].close;
                    const currentPrice = coin.tradePrice;
                    if (prevPrice !== currentPrice) {
                        console.log(prevPrice);
                        this.chart.series[1].data[lastDataIndex].update({
                            x: coin.timestamp,
                            open: coin.openingPrice,
                            high: coin.highPrice,
                            low: coin.lowPrice,
                            close: coin.tradePrice,
                        }, true);
                        console.log(this.chart.series[1].data[lastDataIndex].close);
                        console.log('-------');
                    }
                }
            },
        },
        methods: {
            fetchData: function (coinType) {
                if (coinType) {
                    return axios.get(`https://crix-api-endpoint.upbit.com/v1/crix/candles/days?code=${coinType}&count=180`);
                }
            },
            initializeChart: function (data) {
                this.chart = Highcharts.stockChart('container', {
                    rangeSelector: {
                        selected: 1,
                    },

                    title: {
                        text: this.coinType,
                    },

                    series: [{
                        type: 'candlestick',
                        name: '가격정보',
                        data: data,
                    }],
                });
            },
            clearChart: function () {
                if (this.chart && this.chart.forExport) {
                    this.chart.destroy();
                }
            },
            chartTransaction: function () {
                this.clearChart();
                this.fetchData(this.coinType)
                    .then((result) => {
                        const chartData = result.data
                            .sort((a, b) => {
                                return a.timestamp - b.timestamp;
                            })
                            .map(data => {
                                return {
                                    x: data.timestamp,
                                    open: data.openingPrice,
                                    high: data.highPrice,
                                    low: data.lowPrice,
                                    close: data.tradePrice,
                                };
                            });
                        this.initializeChart(chartData);
                    })
                    .catch(() => {
                        alert('데이터를 가져오는데 실패했습니다.');
                    });
            },
        },
        updated: function () {
            this.chartTransaction();
        },
        mounted: function () {
            this.chartTransaction();
        },
        beforeDestroy: function () {
            this.clearChart();
        },
    };
</script>

<style scoped>

</style>
