# GraphQL

## Creating a GraphQL server using NodeJS, Restify and MongoDB

GraphQL is a query language aiming to replace REST APIs.
GraphQL gives clients the ability to request for exactly what data they need in a single request, which makes data fetching  predictable and fast even on a limited bandwith.


```js
// index.js

const restify = require('restify');
const {openDBConnection} = require('./db.js')
const {logger, setResponseHeaders} = require('./middlewares.js')
const {routes, registerRoutes} = require('./routes.js')

const server = restify.createServer({
  name: 'GraphQL API'
});

server.use(restify.queryParser());
server.use(restify.bodyParser());
server.use(logger);
server.use(setResponseHeaders);

server.on('uncaughtException',function(request, response, route, error){
  console.error(error.stack);
  response.send(error);
});

server.listen(8081, function() {
  console.log('%s listening at %s', server.name, server.url);
  openDBConnection()
});

registerRoutes(server, routes)
```


```js
// middlewares.js

export function setResponseHeaders(req, res, next) {
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.header('Access-Control-Allow-Headers', 'X-Requested-With,content-type, Authorization');
    res.charSet('utf-8');
    next();
}

export function logger(req,res,next) {
  console.log(
    `    time: ${new Date()};
    path: ${req.url};
    method: ${req.method};
    params: ${JSON.stringify(req.params)};
    body: ${req.body};
    `
  );
  next();
}
```

```js
const mongoose = require('mongoose');
mongoose.Promise = global.Promise;

const db = mongoose.connection;
db.on( 'error', console.error.bind( console, 'MongoDB connection error: ') );
db.once( 'open', console.log.bind( console, 'MongoDB connection open') );

const db_URI = 'mongodb://<MY_DB>'

const options = {
  db: { native_parser: true },
  server: { poolSize: 5 },
  replset: { rs_name: 'myReplicaSetName' }
}

export const openDBConnection = () => mongoose.connect(db_URI, options);

export const closeDBConnection = (msg, cb) => {
  const _msg = msg || 'Mongoose connection disconnected'
  mongoose.connection.close(function() {
    console.log(_msg);
    if(cb) cb()
  });
}

process.on('SIGINT', () =>
  closeDBConnection(
    'Mongoose disconnected on app termination',
    _ => process.exit(0)
  )
);
```


```js
// routes.js

const { graphql } = require('graphql')
const { PhotoGraphQLSchema } = require('./PhotoGraphQLSchema.js')

const request = query => graphql( PhotoGraphQLSchema, query )
const handlePromise = function (fn, res) {
  return fn
  .then(data => res.send(data))
  .catch(error => res.send(error))
}

export const routes = [
  {
    verb: 'get',
    path: '/graphql',
    handler: ({query}, res) => handlePromise(request(query.graphql), res)
  },
  {
    verb: 'post',
    path: '/graphql',
    handler: ({body}, res) => handlePromise(request(body), res),
  }
]

export function registerRoutes(server, routesArr) {
  return routesArr.map(r => server[r.verb.toLowerCase()](r.path, r.handler))
}
```

```js
// PhotoGraphQLSchema.js

const {
    graphql,
    GraphQLSchema,
    GraphQLObjectType,
    GraphQLString,
    GraphQLNonNull,
    GraphQLList,
    GraphQLInt
} = require('graphql')

const { find } = require('./controllers.js')

const PhotoType = new GraphQLObjectType({
  name: 'Photo',
  description: 'Single Photo schema',
  fields: () => ({
    title: {
      type: new GraphQLNonNull (GraphQLString),
      description: 'The title of the photo',
    },
    url: {
      type: new GraphQLNonNull (GraphQLString),
      description: 'The url of the photo',
    },
    id: {
      type: new GraphQLNonNull (GraphQLString),
      description: 'The id of the photo',
    },
    created_at: {
      type: new GraphQLNonNull (GraphQLString),
      description: 'Date when the photo was created'
    }
  }),
});


const rootType = new GraphQLObjectType({
  name: 'Root',
  fields: () => ({
    photos: {
      type: new GraphQLList(PhotoType),
      args: {
        id: {
          description: 'If omitted replies with all the photos, if provided it returns the photo with this id',
          type: GraphQLString
        }
      },
      resolve: id => id ? find({ id }) : find({})
    },
  })
});

export const PhotoGraphQLSchema = new GraphQLSchema({
  query : rootType
});
```

```js
// controllers.js

const {Photo} = require('./photo-model.js')

const handlePromise = function ( fn ) {
  return fn
  .then(data => data)
  .catch(error => error)
};

export const find = query => handlePromise(Photo.find(query))
```

```js
// model.js

const mongoose = require('mongoose')
const  {PhotoSchema} = require('../PhotoMongooseSchema.js')

export const Photo = mongoose.model('Photo', PhotoSchema);
```

```js
// PhotoMongooseSchema.js

const mongoose = require('mongoose');
const Schema = mongoose.Schema;

export const PhotoSchema = new Schema({
  title: String,
  url: { type:String, default: ''},
  media_url: { type:String, default: ''},
  username: { type:String, default: ''},
  updated_at: { type: Date, default: Date.now },
  created_at: { type: Date, default: Date.now }
});
```
