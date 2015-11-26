# rails-es6-demo

This demo is following from the [Getting Started with Rails](http://guides.rubyonrails.org/getting_started.html) on the official rubyonrails website, in other word, *I don't know Rails*.

One thing bothers me when I start to try Rails is that it generate Coffeescript by default.

I haven't written Coffeescript for a while after switching to ES6 world. So I was wondering if I could use ES6 in a Rails project.

So I searched for a while on Google and find the first solution.

**1.** Using Sprockets with `sprockets-es6` gem.

In your *Gemfile*, simply add

```
gem "sprockets"
gem "sprockets-es6"
```

And then you can start to write ES6 code in your `app/assets/javascripts` folder. Here the `welcome.es6` for example.

```JavaScript
let square = (x) => x * x
console.log(square(2))
```

It works. However, if we want to use the module system in ES6, like the `require` style form commonjs world or `import` from ES6, it doesn't work.

So If we try to import a  a `test.es6` in the same directory

```JavaScript
import square from './test'
console.log(square(2))
```

And `test.es6` is as simple as `export default test = (x) => x * x`

The compiled version will be

```JavaScript
'use strict';

function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { 'default': obj }; }

var _test = require('./test');

var _test2 = _interopRequireDefault(_test);

console.log((0, _test2['default'])(2));
```

And if we run it in the browser, it will fail because `require` is not defined in the browser.

ref: [Start using ES6 with Rails today](http://blog.arkency.com/2015/05/start-using-es6-with-rails-today/)

Note: I also find this post interesting after I find my second solution. [Using ES6 with Asset Pipeline on Ruby on Rails](http://nandovieira.com/using-es6-with-asset-pipeline-on-ruby-on-rails) It's still using `sprockets` together with gem and I'll discuss it later.

**2.** Using babel that I'm more familiar with(yes, I've looked into it and even made a moudle [babel-jsxgettext](https://github.com/fraserxu/babel-jsxgettext) using its internal parser `babylon`(with better support for JSX)) directly instead of something that I don't know well(GEM).

I checked the code that Rails use to compile Coffeescirpt to JS here which is as simple as using the Coffeescript compiler somehow and transfer it into JavaScript code. https://github.com/rails/coffee-rails/blob/master/lib/rails/generators/coffee/assets/assets_generator.rb#L9

```Ruby
def copy_coffee
  template "javascript.coffee", File.join('app/assets/javascripts', class_path, "#{file_name}.coffee")
end
```

I can do the same with JavaScript!

Here's the workflow.

1. Create a folder named `babel` or whatever else and put a file there with something like `welcome.js`, and yes, I don't like `.es6` extension...

2. Using run `babel babel/welcome.js -o javascript/welcome.js`. Make sure you have Bable install globally `$ npm install -g babel-cli`

3. If you want live compile when you change your code, just add `-w` to watch.

4. If you want to watch a folder like here for a rails project, `babel babel --out-dir javascripts`

**Oh, wait!!! `import` and `require` still doesn't work!**

Don't worry, it's JavaScript, let's figure it out.

So what we will do here is to use `browserify` to make our code work with commonjs `require` and `babelify` for ES6 code.

Make sure you have those packages install as well.

```
npm install --save-dev babel-preset-es2015 babelify browserify
```

Here's the command.

```
browserify babel/welcome.js > javascripts/welcome.js -t [ babelify --presets [ es2015 ] ]
```

This will use the `babelify` transformer together with `es2015` presets to make sure it works for ES6 and bundle all the code together.

In case you are not familar with browserify, [here's the book](https://github.com/substack/browserify-handbook).

Check the code I have in the `app/assets/babel` folder for example. It will print `Bot: yo` if you check the console.

Here's the `pros` and `cons` for these two solutions.

features | GEM Way  | Node way
--------- | ------------- | -------------
ES6 features | ✅  | ✅
Modules system | ❎  | ✅
Compatibility with exisiting code | ✅ | ✅

Both of these two solution support well with legacy code base since it keep the existing code runnig and let people to choose what code they prefer.

For long term consideration(better seperate with front-end code and back-end code), the latter might be more like the modern approach, which embrace us to a better way for front-end engineering(single page for example).
