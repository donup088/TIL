### 데이터를 받아올 동안 로딩구현
- 이벤트 버스로 구현
- Spinner 컴포넌트 만들기
    ```
    <template>
    <div class="lds-facebook" v-if="loading">
        <div></div>
        <div></div>
        <div></div>
    </div>
    </template>

    <script>
    export default {
        props: {
            loading: {
                type: Boolean,
                required: true,
            }
        }
    }
    </script>

    <style>
    .lds-facebook {
        display: inline-block;
        position: absolute;
        width: 64px;
        height: 64px;
        top: 47%;
        left: 47%;
    }

    .lds-facebook div {
        display: inline-block;
        position: absolute;
        left: 6px;
        width: 13px;
        background: #42b883;
        animation: lds-facebook 1.2s cubic-bezier(0, 0.5, 0.5, 1) infinite;
    }

    .lds-facebook div:nth-child(1) {
        left: 6px;
        animation-delay: -0.24s;
    }

    .lds-facebook div:nth-child(2) {
        left: 26px;
        animation-delay: -0.12s;
    }

    .lds-facebook div:nth-child(3) {
        left: 45px;
        animation-delay: 0;
    }

    @keyframes lds-facebook {
        0% {
            top: 6px;
            height: 51px;
        }

        50%,
        100% {
            top: 19px;
            height: 26px;
        }

    }
    </style>
    ```
- bus.js 생성
```
import Vue from 'vue';

export default new Vue();
```
- App.vue와 사용하는 컴포넌트에 적용
```
//NewView.vue
export default {
    components: {
        ListItem
    },
    created() {
        bus.$emit('start:spinner');
        setTimeout(() => {
            this.$store.dispatch('FETCH_NEWS')
                .then(() => {
                    console.log('fetched');
                    bus.$emit('end:spinner');
                })
                .catch((error) => {
                    console.log(error);
                })

        }, 3000);
    }
}
//App.Vue
export default {
    components: {
        ToolBar,
        Spinner
    },
    data() {
        return {
            loadingStatus: false
        };
    },
    methods: {
        startSpinner() {
            this.loadingStatus = true;
        },
        endSpinner() {
            this.loadingStatus = false;
        }
    },
    created() {
        bus.$on('start:spinner', this.startSpinner);
        bus.$on('end:spinner', this.endSpinner);
    },
    beforeDestroy() {
        bus.$off('start:spinner', this.startSpinner);
        bus.$off('end:spinner', this.endSpinner);
    }
}
```