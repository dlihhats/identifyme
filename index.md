In this project, we're going to look at the problem of user identification by their behaviour on the Internet. In particular, we're planning to analyse websites sequences from sessions belonging to various people. Similar problems are being solved in Google Analytics and described in articles on the topics of ["Traversal Pattern Mining"](https://scholar.google.co.uk/scholar?q=traversal+pattern+mining) and ["Sequential Pattern Mining"](https://scholar.google.co.uk/scholar?q=sequential+pattern+mining).

We used the data from an article [A Tool for Classification of Sequential Data](http://ceur-ws.org/Vol-1703/paper12.pdf), but our approach to classification will be different compared with the one used in the article.

The data comes from proxy-servers of the Blaise Pascal University and represents user ID, timestamp, and a visited URL.

### Data Visualisation
Let's check an assumption that our users browse the Internet in different times of the day throughout the week.

```python
layout = go.Layout(
    title="Percentage of all user's sessions",
    height = 1300
)  

fig = tools.make_subplots(rows=5, cols=2, \
    subplot_titles=(sorted(id_name_dict.values())), \
    print_grid=False, \
    specs=[[{}, {}], [{}, {}], [{}, {}], [{}, {}], [{}, {}]], \
    horizontal_spacing = 0.01, vertical_spacing = 0.07)

user_names = sorted(id_name_dict.values(), reverse=True)

for x, y in itertools.product(range(1, 6), range(1, 3)):
    if len(user_names):
        user_name = user_names.pop()

        userdata = new_features_10users\
                    [new_features_10users.target==user_name]\
                    [["day_of_week", "daytime"]].\
                    groupby(["day_of_week", "daytime"]).size().reset_index()
        matrix = []
        totalsessions = sum(userdata[0])
        for dtime in range(0,3):
            weekrow = []
            for day in range(0,7):
                value = userdata[(userdata.day_of_week == day) & \
                        (userdata.daytime == dtime)][0]
                if len(value):
                    weekrow.append(round(float(value) / \
                            totalsessions * 100, 1))
                else:
                    weekrow.append(0)
            matrix.append(weekrow)
        
        if y == 1:
            y_lable=['Morning', 'Afternoon', 'Evening']
        else:
            y_lable=[]
        
        data = go.Heatmap(
            z=matrix,
            x=['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su'],
            y=y_lable,
            showscale=False,
            zsmooth="best"
        )

        fig.append_trace(data, x, y)
        
fig['layout'].update(layout)

iplot(fig)
```

<iframe width="100%" height="1200" frameborder="0" scrolling="no" src="https://plot.ly/~dlihhats/7.embed"></iframe>


Let's take a look at the average percentage of time spent by each user on Facebook, Youtube and Top30 websites within a session.

```python
trace1 = go.Bar(
    x=sorted(id_name_dict.values()),
    y=[round(np.mean(new_features_10users[(new_features_10users.target==user) & \
              (new_features_10users.fb_portion > 0)]["fb_portion"]) * 100, 1) \
                                for user in sorted(id_name_dict.values())],
    name='Facebook'
)
trace2 = go.Bar(
    x=sorted(id_name_dict.values()),
    y=[round(np.mean(new_features_10users[(new_features_10users.target==user)& \
              (new_features_10users.youtube_portion > 0)]["youtube_portion"]) * 100, 1) \
                                 for user in sorted(id_name_dict.values())],
    name='Youtube'
)

trace3 = go.Scatter(
    x=sorted(id_name_dict.values()),
    y=[round(np.mean(new_features_10users[(new_features_10users.target==user) & \
              (new_features_10users.top30_portion > 0)]["top30_portion"]) * 100, 1) \
                                 for user in sorted(id_name_dict.values())],
    name="Top-30 Websites"
)

data = [trace1, trace2, trace3]
layout = go.Layout(
    title="Average % of time spent on websites within a session",
    barmode='group',
    yaxis=dict(
        title='% of time in session'
        )
)

fig = go.Figure(data=data, layout=layout)
iplot(fig)
```

<iframe width="100%" height="400" frameborder="0" scrolling="no" src="https://plot.ly/~dlihhats/1.embed"></iframe>
