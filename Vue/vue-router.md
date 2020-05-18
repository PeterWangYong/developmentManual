```
npm install vue-router
```

```html
<router-link to="/user/123"></router-link>
<router-link :to="{name: 'user', params: {id: 123}}"></router-link>
<router-view></router-view>
```

```js
this.$route
this.$router
this.$router.push()
this.$router.replace()
this.$router.go()
```

```js
routes: [
	{ 
		path: '/home',
		name: 'home'
		component: 'xxx',
		children: [
			{
				path: 'page',
				component: 'xxx',
				alias: '/page'   // /page == /home/page
			}
		],
		beforeEnter((to, from, next) => {}),
		meta: {requireAuth: true}
	},
	{ 
		path: '/user/:id'   // this.$route.params.id,
		component: () => import('../xxx')
	},
	{
		path: '*',  			// this.$route.params.pathMatch
		component: '404'
	},
	{
		path: '/food/:id',
		components: {	   
			a: 'xxx',			// <router-view name="a"/>
			b: 'xxx',			// <router-view name="b"/>
		},
		props: {
			a: true,     // component: <template><div>{{id}}</div></template>  
			b: false		 // <script>export 	default { props: ['id']}
		}
	},
	{
		path: '/unknown',
		redirect: '/home',   // redirect: {name: 'home'}   redirect: to => {return '/home'}
	}
]
```

```js
router.beforeEach((to, from, next) => {
	next()
	// next('/')
	// next({name: 'home'})
	// next(false)
	// next(error)
})

router.beforeResolve((to, from, next) => {})
router.afterEach((to, from) => {})

<template></template>
<script>
export default {
	beforeRouteEnter(to, from, next) {},
	beforeRouteUpdate(to, from, next) {},
	beforeRouteLeave(to, from, next) {}
}
```

```html
<transition>
	<router-view></router-view>
</transition>

<template>
	<transition name="slide">
  	<div>
    </div>
  </transition>
</template>
```