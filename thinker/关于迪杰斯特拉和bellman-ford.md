# 关于迪杰斯特拉和bellman-ford的思考
> 空想主义是真的不行，之前都没意识到迪杰斯特拉算法的本质，之前一直记图、记代码。题型一变，时间一久，什么也记不住了。该不会还是不会。

## 共同点
> 从起点a出发，如果a到b的距离是整个图中a出发的到其他n-1个点中最短的距离，则不可能存在点c，则不可能存在点c使a->c->b使a到b的距离更短。因此，a->b的距离已经确定

## 迪杰斯特拉(适用于非负单源)
### 本质
> 迪杰斯特拉算法其实就是一个贪心的过程，每次去访问没有访问节点中距离最短的点。直到所有能访问的点被访问。

### 过程
> 从未访问点集合中找到一个离出发点a最小的点。(这个点的距离可以确定下来不会改变了，因为已经是最短的了。) 标记这个点为已访问，从这个点出发，去更新这个点的邻居到a点的距离。
```
	int dis[n];
	bool visit[n];
	memset(dis, inf, sizeof(dis));
	memset(visit, false, sizeof(visit));
	dis[src] = 0; // 标记起点为0
	while(true){
		int u = -1;
		int m = inf;
		for(int j = 0; j < n; j++){
			if(j != src && !visit[j] && dis[j] < m){
				u = j;
				m = dis[j];
			}
		}
		if(u == -1)
			break; // 全部能访问的点都访问完了。
	    visit[u] = true; // 因为这个点是最小的点，所以这个点确定下来了。之后不会有比他更小的路径。
	    for(int v = 0; v < n; ++v){
		    if(map[u][v] != unconect && dis[u] + map[u][v] < dis[v]) // u可达v
		    dis[v] = dis[u] + map[u][v];
		}		
	}
```
### 复杂度分析
> 假设n个点都可达，那我们外层的循环需要执行n次，因为每次只选择一个点。然后内循环还需要去找最小的边，需要遍历所有的点，即需要O(N*(2N)),如果是邻接表的话，则需要O（N*(N+E))

> 我们可以优化一下这个程序，如果在选取最小的点时候能够直接拿到，而不需要遍历，这个时候我们就需要一个优先队列，小顶堆，保证每次堆顶都是最小距离的顶点即可。这时候循环里面就变成了logn，所以总的应该是n*logn(用邻接表保存信息)
```
// 定义一种数据结构来存储顶点以及出发点到顶点的距离。
struct Item{
  int node; // 保存节点左边。
  int val; // 保存距离出发点的距离， 
  bool operator <(Item a) const { return val < a.val;}
  bool operator >(Item a) const { return val > a.val;}  
  Item(int n, int v) : node(n), val(v){}
};
// 把当前访问节点的邻居加入到优先队列中。
void get_neighbor(priority_queue<Item, vector<Item>, greater<Item>> &q, int (*map)[200], int k, int temp, int N){
        for(int i = 0; i < N; i++)
            if(i != k && map[k][i] < unconect)
                q.push(Item(i, temp + map[k][i]));
}
 int map[N][200];
 priority_queue<Item, vector<Item>, greater<Item>> q;
 get_neighbor(q, map, k, 0, N); // 先访问出发点
 Item node(1,2);
 visit[k] = true;
 time[k] = 0;
 while(!q.empty()){
            node = q.top(); // 找到距离出发点最短的点。
            q.pop();
            int index = node.node;
            int val = node.val;
            if (!visit[index]){
                 cnt = cnt + 1;
                 visit[index] = true;
            }else
                continue;
            if(val > time[index]) // 队列的距离更大。那就不考虑了
                continue;
            else{
                time[index]=val;
                get_neighbor(q, map, index, val, N); //去把他的邻居放到队列中
            }
 }
 // 跳出循环后，time储存着距离出发点的所有点。   
```

## bellman-ford
>  这个算法就是，我们有n个点，那我就去访问n-1次，每次访问遍历所有的边，然后更新可以更新的距离，这样在最坏的情况下，一次是可以确定一个点的距离的，下一次会确定上次确定点的下一个点的距离。 当不再发生更新的时候，也可以直接break了。这样相当于在某次循环的时候确定了几个点的距离。

### 用处
> 他的时间复杂度就比较高了，是O(N*E)，当边特别多的时候就很鸡肋，但是他的好处是他可以检测是否存在负的权重。而迪杰斯特拉只能用于非负权重的图中。

```
        int dis[N];
        for(int i = 0; i <  N; i++){
            dis[i] = 1000000;
            visit[i] = false;
        }
        dis[K-1] = 0; // 到自己的时间为0
        for(int i = 0; i < N - 1; i++){ // 对所有的边松弛n-1次，
            auto begin = times.begin();
            bool flag = true;
                while(begin != times.end()){
                    int u = (*begin)[0] - 1;
                    int v = (*begin)[1] - 1;
                    int w = (*begin)[2];
                    if(dis[u] + w < dis[v]){
                        flag = false;
                        dis[v] = dis[u] + w; // 有中间点u使路径更短，则更新。
                        if(visit[v] == false){ // 之前未访问到。
                            visit[v] = true;
                            cnt = cnt + 1;
                        }
                    }
                    begin++;
                }
           if(flag)
               break; //说明没有再更新过，以后也不会更新了。
        }
        for(int i = 0; i < N; i++){
            if(dis[i] == 1000000)
                return -1;
            else
                s = max(s, dis[i]);
        }
```
