### mixin 사용
- 컴포넌트에서 중복되는 코드를 없앨 수 있다.
```
import bus from '../utils/bus.js';

export default{
    created() {
        bus.$emit('start:spinner');

        this.$store.dispatch('FETCH_LIST',this.$route.name)
            .then(() => {
                console.log('fetched');
                bus.$emit('end:spinner');
            })
            .catch((error) => {
                console.log(error);
            })

    }
}
```
```
export default {
    components: {
        ListItem
    },
    mixins: [ListMixin]
}
```