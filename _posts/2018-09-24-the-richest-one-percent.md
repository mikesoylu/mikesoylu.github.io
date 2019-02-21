---
layout: post
title: The Richest One Percent
cta: See what happens »
---

The richest one percent owns more than the half of the world's wealth. This statistic seemed unintuitive to me. Why isn't wealth more evenly distributed? So I decided to make a simple simulation to see what happens when you unleash a free market 🤓

The graph below shows the output of the simulation: the wealth of 50 people. Each start off with $100. On every simulation turn, they randomly pair-up and flip a coin. The winner takes $1 from the loser.

If bankruptcy is disabled people with negative wealth get *saved*, possibly by the government and are restored to 0. So they can keep participating in the market — toggle this off to see what happens in more unforgiving circumstances 😅

Even though this models a super simple marketplace where everyone is unrealisticly *equal*, we get some interesting emergent behaviour out of it. Not unlike the real world.

Go ahead and run the simulation and see what happens!

<br>

<button onclick="start()">Run</button>
<button onclick="stop()">Stop</button>
<button onclick="init()">Reset</button>
<label style="font-family: sans-serif; font-size: 16px;"><input type="checkbox" onclick="toggleBankruptcy()"> Enable Bankruptcy</label>

<div id="visualization"></div>

<br>

P.S. Tell me what you think about it on [twitter](https://twitter.com/intent/tweet?text=My thoughts on {{ page.title }} by @mikesoylu&url={{ site.url }}{{ page.url }}){:target="_blank"}!

<script src="https://cdnjs.cloudflare.com/ajax/libs/svg.js/2.6.6/svg.min.js"></script>
<script>
  var NAMES = ["Brandon", "Meagan", "Everett", "Anthony", "Karen", "Hester", "Faye", "Vince", "Corina", "Sang", "Emilia", "Roger", "Zachary", "Ken", "Earl", "Mariano", "Zoe", "Glenn", "Jocelyn", "Olga", "Jae", "Lemuel", "Odis", "Lillian", "Alton", "Carter", "Leslie", "Carlo", "Silas", "Nelda", "Heather", "Willis", "Ronda", "Cecelia", "Edwin", "Darla", "Shad", "Allen", "Rolland", "Mike", "Bertie", "Flora", "Leann", "Stefan", "Derek", "Chris", "Cecilia", "Sabrina", "Cordell", "Lela", "Heidi", "Letha", "Rayford", "Adele", "Marquita", "Shirley", "Augustine", "Lorena", "Jack", "Emmett", "Clarice", "Violet", "Dolores", "Adolph", "Cornelius", "Eugene", "Trina", "Mabel", "Herbert", "Angelique", "Clarence", "Berry", "Elizabeth", "Tyrell", "Bradly", "Darrel", "Dorian", "Mable", "Hipolito", "Lenard", "Will", "Lynnette", "Celia", "Kristopher", "Mac", "Kaitlin", "Aileen", "Howard", "Rebecca", "Lesley", "Domenic", "Therese", "Felicia", "Riley", "Shauna", "Rosalie", "Francisco", "Reyna", "Dana", "Alba"];
  var STEP_SIZE = 250;
  var BET_SIZE = 1;
  var NUM_AGENTS = 50;
  var STARTING_CASH = 100;
  var MAX_CASH = NUM_AGENTS * STARTING_CASH;

  var agents;
  var elapsedTime;
  var simutlating = false;
  var bankruptcyEnabled = false;

  function init() {
    // prepare data
    agents = [];
    elapsedTime = 0;

    for (var i = 0; i < NUM_AGENTS; i++) {
      agents.push({
        name: NAMES[i % NAMES.length],
        cash: STARTING_CASH,
        timeOfDeath: 0,
        color: "#444"
      });
    }
  }

  function go() {
    // helpers
    function bar(agent, y) {
      var color = agent.color,
        name = agent.name,
        cash = Math.floor(agent.cash);

      if (cash === 0) {
        var width = new SVG.Number(0).to("%");

        draw.text(function(add) {
            add.tspan(name);
            add.tspan("$" + cash).dx(5).fill("rgba(255,0,102,0.2)").attr("font-size", 8);
          })
          .move(width.plus("1%"), y - 7)
          .font({ family: "sans-serif", size: 9 })
          .fill("#f06");
      } else {
        var width = new SVG.Number(cash).divide(MAX_CASH + 400).to("%");

        draw.rect(width, 9).y(y).fill(color);
        draw.text(function(add) {
            add.tspan(name)
            add.tspan("$" + cash).dx(5).fill("#999").attr("font-size", 8)
          })
          .move(width.plus("1%"), y - 7)
          .font({ family: "sans-serif", size: 9 })
          .fill("#666");
      }
    }

    function splitAt(i, xs) {
      var a = xs.slice(0, i);
      var b = xs.slice(i, xs.length);
      return [a, b];
    }

    function shuffle(xs) {
      return xs.slice(0).sort(function() {
        return .5 - Math.random();
      });
    }

    function zip(xs) {
      return xs[0].map(function(_,i) {
        return xs.map(function(x) {
          return x[i];
        });
      });
    }

    function step() {
      var taxPool = 0;

      // trade
      var liveAgents = bankruptcyEnabled ? agents.filter(function(a) { return a.cash > 0; }) : agents;
      var pairs = zip(splitAt(liveAgents.length / 2, shuffle(liveAgents)));

      pairs.forEach(function(pair) {
        var coinflip = Math.random() * 2 | 0;
        var otherside = !coinflip | 0;

        // lose
        var loserCash = pair[coinflip].cash;
        pair[coinflip].cash -= Math.min(loserCash, BET_SIZE);

        // win - excluding tax
        pair[otherside].cash += Math.min(loserCash, BET_SIZE);

        // check if loser is dead
        if (pair[coinflip].cash === 0) {
          pair[coinflip].timeOfDeath = elapsedTime;
        }
      });

      // advance time
      elapsedTime++;
    }

    // graphics context
    var draw = SVG("visualization").size("100%", NUM_AGENTS * 10)

    // simulation loop
    function loop() {
      for (var i = 0; i < STEP_SIZE; i++) {
        if (simutlating) step();
      }

      // sort
      agents.sort(function(a, b) {
        if (a.cash || b.cash) {
          return b.cash - a.cash;
        } else {
          return b.timeOfDeath - a.timeOfDeath;
        }
      });

      // draw
      draw.clear();
      agents.forEach(function(agent, i) {
        bar(agent, i * 10)
      });

      requestAnimationFrame(loop);
    }

    // run loop
    init();
    loop();
  }

  function stop() {
    simutlating = false;
  }

  function start() {
    simutlating = true;
  }

  function toggleBankruptcy() {
    bankruptcyEnabled = !bankruptcyEnabled;
  }

  if (document.addEventListener) {
    document.addEventListener("DOMContentLoaded", go, false);
  } else if (document.attachEvent) {
    document.attachEvent("onreadystatechange", go);
  } else {
    window.onload = go;
  }
</script>
