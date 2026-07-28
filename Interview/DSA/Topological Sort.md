# DFS:
Put the element into stack after searching for all its neighbours.
```
void findTopoSort(int node, vector<int>& vis, vector<vector<int>>& adj, stack<int>& st) {
    vis[node] = 1;
    for (int it : adj[node]) {
        if (!vis[it]) {
            findTopoSort(it, vis, adj, st);
        }
    }
    // push the node after all its neighbours
    st.push(node);
}

// Run a for loop in the main function to check for not connected components present as well. This will do the job.
```

# BFS
### Kahns Algorithm:
Reducing the in-degree of the nodes.
Steps to do it:
1. First convert it into adjacency list.
2. Next is to compute in degrees for each one of them and add them to a queue of those whose in-degree is zero. If none present return false;
3. Add everything to a while loop:
   ```
   while(!q.empty()){
            int top = q.front();
            q.pop();
            ans.push_back(top);
            for(int i=0;i<adj[top].size();i++){
                indegree[adj[top][i]]--;
                if(indegree[adj[top][i]]==0)q.push(adj[top][i]);
            }
        }
   ```
4. At any point we can do is:
   ```
   if(ans.size()!=V and q.empty())return false;
   
   // To check if it has a valid topological sort
   ```