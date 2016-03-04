# poker-holdem-engine

`poker-holdem-engine` provides an engine to play Texas Hold'em Poker in respect of the [official rules](https://it.wikipedia.org/wiki/Texas_hold_%27em), with the only exception that no constraints are imposed on the raise amount.

Poker here is meant to be played by other programs, which should be listen for POST http request somewhere in the internet.

## start a tournament

```
let engine = require('poker-holdem-engine');

engine.emit('game:start', {
  tournamentId: 'id_of_the_tournament',
  players: [{
    name: 'r2-d2',
    url: 'http://r2-d2.com/'
  }, {
    name: 'c-3p8',
    url: 'http://c-3po.com/'
  }]
});
```

Players should be object with at least the `name`, and `url` properties specified.

On the specified end point there should be an app, responding on the `POST /bet`, and `GET /version` routes.

Every time something of interesting happen the message `gamestate:updated` is notified, with a parameter containing further information about the state of the game.

## quit a tournament

```
let engine = require('poker-holdem-engine');
engine.emit('game:end');
```

When the tournament finishes the message `tournament-finished` is notified.

## prepare your player

It's possible to code your player in whatever language you want. In the following example i will use javascript.

```
// server.js

'use strict';

const player = require('./player');

const express = require('express');
const http = require('http');
const bodyParser = require('body-parser');

const app = express();
const server = http.Server(app);

app.use(bodyParser.json());

app.get('/', function(req, res) {
  res.sendStatus(200);
});

app.get('/version', function(req, res) {
  res.status(200).send(player.VERSION);
});

app.post('/bet', function(req, res) {
  player.bet(req.body, bet => res.status(200).send(bet.toString()));
});


const port = parseInt(process.env['PORT'] || 1337);

server.listen(port, function() {
  console.log('Server listening on port', server.address().port);
});
```

And the player module

```
exports = module.exports = {

  VERSION: 'pokerstar v1',

  bet: function (gamestate, bet) {

    // gamestate contains info about the state of the game.
    // bet is the function you should use to send your bet.

    // currently we just fold every single hand
    return bet(0);

  }

};
```