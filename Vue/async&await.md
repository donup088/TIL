### async, await 사용
- 코드를 더 직관적으로 바꿀 수 있다.
```
// FETCH_NEWS(context) {
    //     return fethchNewsList()
    //         .then(response => {
    //             console.log(response);
    //             context.commit('SET_NEWS', response.data);
    //             return response;
    //         })
    //         .catch(error => {
    //             console.log(error);
    //         })
    // },
    async FETCH_NEWS(context) {
        const response = await fethchNewsList();
        context.commit('SET_NEWS', response.data);
        return response;
    },
```
- api를 사용하는 곳에서 try-catch를 사용하여 error처리를 해준다.
```
function fetchList(pageName) {
    try {
        return axios.get((`${config.baseUrl}${pageName}/1.json`))
    }
    catch (error) {
    console.log(error);
    }
}
```