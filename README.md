# React Native RefreshableListView
A pull-to-refresh ListView which shows a loading spinner while your data reloads

In action (from [ReactNativeHackerNews](https://github.com/jsdf/ReactNativeHackerNews)):

![React Native Hacker News](http://i.imgur.com/gVmrxDe.png)

## Usage

### Example

```js
var React = require('react-native')
var {Text, View, ListView} = React
var RefreshableListView = require('react-native-refreshable-listview')

var ArticleStore = require('../stores/ArticleStore')
var StoreWatchMixin = require('./StoreWatchMixin')
var ArticleView = require('./ArticleView')

var ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2}) // assumes immutable objects

var ArticlesScreen = React.createClass({
  mixins: [StoreWatchMixin],
  getInitialState() {
    return {dataSource: ds.cloneWithRows(ArticleStore.all())}
  },
  getStoreWatches() {
    this.watchStore(ArticleStore, () => {
      this.setState({dataSource: ds.cloneWithRows(ArticleStore.all())})
    })
  },
  reloadArticles() {
    // returns a Promise of reload completion
    // for a Promise-free version see ControlledRefreshableListView below
    return ArticleStore.reload()
  },
  renderArticle(article) {
    return <ArticleView article={article} />
  },
  render() {
    return (
      <RefreshableListView
        dataSource={this.state.dataSource}
        renderRow={this.renderArticle}
        loadData={this.reloadArticles}
        refreshDescription="Refreshing articles"
      />
    )
  }
})
```

### RefreshableListView
Replace a ListView with a RefreshableListView to add pulldown-to-refresh 
functionality. Accepts the same props as ListView (except `renderHeader`, see below), with a few extras.

#### Props

- `loadData: func.isRequired`
  A function returning a Promise or taking a callback, invoked upon pulldown. 
  The refreshing indicator (spinner) will show until the Promise resolves or the callback 
  is called.
- `refreshDescription: oneOfType([string, element])`
  Text/element to show alongside spinner. If a custom 
  `refreshingIndicatorComponent` is used this value will be passed as its 
  `description` prop.
- `refreshingIndicatorComponent: oneOfType([func, element])`
  Content to show in list header when refreshing. Can be a component class or 
  instantiated element. Defaults to `RefreshableListView.RefreshingIndicator`.
  You can easily customise the appearance of the indicator by passing in a
  customised `<RefreshableListView.RefreshingIndicator />`, or provide your own
  entirely custom content to be displayed.
- `minDisplayTime: number`
  Minimum time the spinner will show for.
- `minBetweenTime: number`
  Minimum time after a refresh before another refresh can be performed.
- `minPulldownDistance: number`
  Minimum distance (in px) which the list has to be scrolled off the top to 
  trigger a refresh.
- `ignoreInertialScroll: bool`
  Require the user to be actually touching the screen when the pulldown distance 
  exceeds `minPulldownDistance` to trigger a refresh (eg. not just inertially 
  scrolling off the top). Defaults to `true`.
- `onScroll: func`
  An event handler for the `onScroll` event which will be chained after the one
  defined by the `RefreshableListView`.
- `scrollEventThrottle: number`
  How often `ListView` produces scroll events, in ms. Defaults to a fairly low 
  value, try setting it higher if you encounter performance issues. Keep in mind
  that a higher value will make the pulldown-to-refresh behaviour less responsive.

### ControlledRefreshableListView
Low level component used by `RefreshableListView`. Use this directly if you want 
to manually control the refreshing status (rather than using a Promise).

This component is more suitable for use in a [Redux](https://github.com/rackt/redux)-style connected component.

```js
var React = require('react-native')
var {Text, View, ListView} = React
var {ControlledRefreshableListView} = require('react-native-refreshable-listview')

var ArticleView = require('./ArticleView')

var ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2}) // assumes immutable objects

var ArticlesScreen = React.createClass({
  propTypes: {
    // eg. props mapped from store state
    articles: React.PropTypes.array.isRequired,
    isRefreshingArticles: React.PropTypes.bool.isRequired,
    // eg. a bound action creator
    refreshArticles: React.PropTypes.func.isRequired,
  },
  getInitialState() {
    return {dataSource: ds.cloneWithRows(this.props.articles)}
  },
  componentWillReceiveProps(nextProps) {
    if (this.props.articles !== nextProps.articles) {
      this.setState({dataSource: ds.cloneWithRows(nextProps.articles)})
    }
  },
  renderArticle(article) {
    return <ArticleView article={article} />
  },
  render() {
    return (
      <ControlledRefreshableListView
        dataSource={this.state.dataSource}
        renderRow={this.renderArticle}
        isRefreshing={this.props.isRefreshingArticles}
        onRefresh={this.props.refreshArticles}
        refreshDescription="Refreshing articles"
      />
    )
  }
})
```

#### Props
- `onRefresh: func.isRequired`
  Called when user pulls listview down to refresh.
- `isRefreshing: bool.isRequired`
  Whether or not to show the refreshing indicator.
- `refreshDescription: oneOfType([string, element])`
  *See `RefreshableListView`*
- `refreshingIndicatorComponent: oneOfType([func, element])`
  *See `RefreshableListView`*
- `minPulldownDistance: number`
  *See `RefreshableListView`*
- `ignoreInertialScroll: bool`
  *See `RefreshableListView`*
- `onScroll: func`
  *See `RefreshableListView`*
- `renderHeaderWrapper: func`
  *See `RefreshableListView`*
- `renderHeader: func`
  **Deprecated** - use `renderHeaderWrapper` instead.
- `scrollEventThrottle: number`
  *See `RefreshableListView`*

### RefreshableListView.RefreshingIndicator
Component with activity indicator to be displayed in list header when refreshing.
(also exposed as `ControlledRefreshableListView.RefreshingIndicator`)

#### Props

- `description: oneOfType([string, element])`
  Text/element to show alongside spinner.
- `stylesheet: object`
  A stylesheet object which overrides one or more of the styles defined in the 
  [RefreshingIndicator stylesheet](lib/RefreshingIndicator.js).
- `activityIndicatorComponent: oneOfType([func, element])`
  The spinner to display. Defaults to `<ActivityIndicatorIOS />`.

### RefreshableListView.DataSource, ControlledRefreshableListView.DataSource
Aliases of `ListView.DataSource`, for convenience.

## Customising the refresh indicator (spinner)

### Style the default RefreshingIndicator
```js
var indicatorStylesheet = StyleSheet.create({
  wrapper: {
    backgroundColor: 'red',
    height: 60,
    marginTop: 10,
  },
  content: {
    backgroundColor: 'blue',
    marginTop: 10,
    height: 60,
  },
})

<RefreshableListView
  refreshingIndicatorComponent={
    <RefreshableListView.RefreshingIndicator stylesheet={indicatorStylesheet} />
  }
/>
```

### Provide a custom RefreshingIndicator

```js
var MyRefreshingIndicator = React.createClass({
  render() {
    return (
      <View>
        <MySpinner />
        <Text>{this.props.description}</Text>
      </View>
    )
  },
})

<RefreshableListView refreshingIndicatorComponent={MyRefreshingIndicator} />
// or
<RefreshableListView refreshingIndicatorComponent={<MyRefreshingIndicator />} />
```

### changelog

- **2.0.0-beta3**
  - fixed proptype warnings from internal component
  - adjusted default styling of refreshing indicator

- **2.0.0-beta2**
  - pulling down now reveals refreshing indicator behind current view, rather
  than the refreshing indicator moving down from the top of the screen
  - `renderHeaderWrapper` is no longer used 
  - fixed infinite refreshing when pulled down

- **1.2.0**
  - deprecated `renderHeader` in favour of `renderHeaderWrapper` as some 
  developers seemed to be confused by the fact that a `renderHeader` handler 
  for a standard `ListView` will not automatically *just work* with this component,
  but rather needs to be modified as described in the documentation. The new prop
  `renderHeaderWrapper` works identically to the previous one, however hopefully
  now it is named differently it will be more apparent that its behaviour is not
  the same as with `ListView`. The `renderHeader` prop will be removed in 2.0.
- **1.1.0**
  - added behaviour to ignore inertial scrolling (@dhrrgn)
  - exposed props: ignoreInertialScroll, scrollEventThrottle
- **1.0.0**
  - Split RefreshableListView into 3 parts: 
    - RefreshableListView handles 'refreshing' state by invoking 'loadData' 
      callback and waiting for resolution.
    - ControlledRefreshableListView handles rendering of ListView header, 
      depending on isRefreshing prop. Calls onRefresh handler when 
      pulldown-to-refresh scroll motion occurs.
    - RefreshingIndicator is the component rendered in the header of the 
      ListView when refreshing. Pass in a customised version of this (or a 
      completely different component) to RefreshableListView or 
      ControlledRefreshableListView if you want to customise refresh indicator 
      appearance.
  - Added Jest unit tests
- **0.3.0** added minPulldownTime & minBetweenTime props, fixed bug where 
  refresh could happen twice
- **0.2.0** added support for ListView props setNativeProps and 
  getScrollResponder (@almost & @sscotth)
