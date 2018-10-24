### 1. $ create-react-app my-ssr-app
### 2. $ cd my-ssr-app
### 3. $ yarn start

### 4. src/Home.js
```javascript
import React from 'react';

export default props => {
  return <h1>Hello {props.name}!</h1>;
};
```

### 5. src/App.js
```javascript
import React from 'react';
import Home from './Home';

export default () => {
  return <Home name="Alligator" />;
};
```

### 6. src/index.js
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.hydrate(<App />, document.getElementById('root'));
```

### 7. yarn add express
### 8. mkdir server && touch server/index.js
### 9. server/index.js
```javascript
import path from 'path';
import fs from 'fs';

import React from 'react';
import express from 'express';
import ReactDOMServer from 'react-dom/server';

import App from '../src/App';

const PORT = process.env.PORT || 3006;
const app = express();

app.use(express.static('./build'));

app.get('/*', (req, res) => {
  const app = ReactDOMServer.renderToString(<App />);

  const indexFile = path.resolve('./build/index.html');
  fs.readFile(indexFile, 'utf8', (err, data) => {
    if (err) {
      console.error('Something went wrong:', err);
      return res.status(500).send('Oops, better luck next time!');
    }

    return res.send(
      data.replace('<div id="root"></div>', `<div id="root">${app}</div>`)
    );
  });
});

app.listen(PORT, () => {
  console.log(`Server is listening on port ${PORT}`);
});
```

### 10. yarn add webpack@4.19.1 webpack-cli babel-core babel-loader
babel-preset-env babel-preset-react-app nodemon webpack-node-externals npm-run-all --dev

### 11. .babelrc
```javascript
{ "presets": ["@babel/preset-env", "react-app"] }
```

### 12. webpack.server.js
```javascript
const path = require('path');
const nodeExternals = require('webpack-node-externals');

module.exports = {
  entry: './server/index.js',

  target: 'node',

  externals: [nodeExternals()],

  output: {
    path: path.resolve('server-build'),
    filename: 'index.js'
  },

  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader'
      }
    ]
  }
};
```

### 13. package.json
```javascript
"scripts": {
  "dev:build-server": "NODE_ENV=development webpack --config ./webpack.server.js --mode=development -w",
  "dev:start": "nodemon ./server-build/index.js",
  "dev": "npm-run-all --parallel build dev:*"
},
```
### 14. yarn run dev