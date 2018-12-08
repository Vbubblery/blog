# Next.js (Server side render)
## Problem and Solution
###  P1- Did not expect server HTML to contain a \<div\> in \<div\>.
<span style="color:red">The problem description: the render result between the server side and the client is different.</span>

**Solution: Create a new Component for React-FullPage and then use the lazy-loading module to load it at the same times, disable the SSR option.**

A new component:


```
class Fullpage extends React.Component {
  constructor(props) {
    super(props);
  }
  render(){
    return(
    <ReactFullpage
      {...fullpageOptions}
      render={()=>{
        return(
          <ReactFullpage.Wrapper>
            <div className="section">1</div>
            <div className="section">2</div>
          </ReactFullpage.Wrapper>
        )
      }}
      />)
  }
}
```

load it lazyly and disable SSR:

```jsx
const DynamicComponentWithoutSSR = dynamic(() => import('../components/Fullpage'),{ssr: false})
```
