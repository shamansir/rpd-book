# Code Examples

All the examples below are _interactive_—you are free to connect Nodes and manipulate the Patch Flow.

To **connect** Outlet of one Node to Inlet of another, use the _click the Outlet_ → _move your mouse_ → _click the Inlet_ method.

<img src="./assets/rpd-connect.gif" width="320px" />

To **disconnect**, _click the Inlet_ → _move your mouse_ → _click anywhere_.

<img src="./assets/rpd-disconnect.gif" width="320px" />

To **reconnect** to another Inlet: _click currently connected Inlet_ → _move your mouse_ → _click wanted Inlet_.

<img src="./assets/rpd-reconnect.gif" width="320px" />

<!--
> For the moment, it could be hard to connect Nodes using Touch control on a small screens or with high sensitivity setup, but [Touch Support is planned](https://github.com/shamansir/rpd/issues/368). -->

Examples with source code:

* [Random Generator](#random-generator)
* [Flag Generator](#flag-generator)
* [HTML5 Canvas and Custom Toolkit](#html5-canvas-and-custom-toolkit)

## Random Generator

Random Generator with the help of [`util`](https://github.com/shamansir/rpd/tree/master/src/toolkit/util) toolkit. Following a signal of Metronome, which "bangs" every 3 seconds by default, Random Generator yields a new value between specified minimum and maximum (here: a number from 0 to 26, just for the sake of the example). The result then goes to a multiply-by-two Node, which is produced from a new type defined in the code. Then we Log last five generated random values and last five multiplied value, just to keep track on things.

<div id="example-one"></div>

```javascript
Rpd.renderNext('svg', document.getElementById('example-one'),
               { style: 'compact-v' });

var patch = Rpd.addPatch('Generate Random Numbers').resizeCanvas(800, 110);

// add Metro Node, it may generate `bang` signal with the requested time interval
var metroNode = patch.addNode('util/metro', 'Metro').move(40, 10);

// add Random Generator Node that will generate random numbers on every `bang` signal
var randomGenNode = patch.addNode('util/random', 'Random').move(130, 20);
randomGenNode.inlets['max'].receive(26); // set maximum value of the generated numbers

// add Log Node, which will log last results of the Random Generator Node
var logRandomNode = patch.addNode('util/log', 'Log').move(210, 60);
randomGenNode.outlets['out'].connect(logRandomNode.inlets['what']);

// define the type of the node which multiplies the incoming value by two
var multiplyTwoNode = patch.addNode('core/basic', '* 2', {
    process: function(inlets) {
        return {
            'result': (inlets.multiplier || 0) * 2
        }
    }
}).move(240, 10);
var multiplierInlet = multiplyTwoNode.addInlet('util/number', 'multiplier');
var resultOutlet = multiplyTwoNode.addOutlet('util/number', 'result');

// connect Random Generator output to the multiplying node
var logMultiplyNode = patch.addNode('util/log', 'Log').move(370, 20);
resultOutlet.connect(logMultiplyNode.inlets['what']);

// connect Random Generator output to the multiplying node
randomGenNode.outlets['out'].connect(multiplierInlet);

// finally connect Metro node to Random Generator, so the sequence starts
metroNode.outlets['out'].connect(randomGenNode.inlets['bang']);
```

## Flag Generator

Tries to generate Country Flag from Unicode codepoints, [inspired by article "Emoji flags from ISO 3166-1 country codes"](https://bendodson.com/weblog/2016/04/26/emoji-flags-from-iso-3166-country-codes-in-swift). (There's also ["Emoji Flag Redux"](https://esham.io/2015/04/emoji-flags-redux) article, written earlier). When you see blank squares (_tofu_) in "Maybe Flag" Node, there's no flag for this combination of letters.

Both Metronomes tick with different period—top one waits two seconds to pass between ticks, bottom one waits three. Ticks trigger corresponding Random Generators to generate a new value between 0 and 26 (so, 0 <= value < 26), which is used as an index of a letter in the alphabet to yield from Letter nodes. Then, two letters are sent to `Maybe<Flag>` Node where they are combined, and if they form a proper two-letter ISO code of the country, you see the flag. If not, you see two _tofu_ squares or a squares with letters inside. Or, if your system is not supporting Emoji, you see something else and then for you this one is not the kind of representative example.

<div id="example-two"></div>

```javascript
Rpd.renderNext('svg', document.getElementById('example-two'),
                      { style: 'compact-v' });

var patch = Rpd.addPatch('Flag Generator').resizeCanvas(800, 200);

var metro1 = patch.addNode('util/metro').move(50, 30);
var metro2 = patch.addNode('util/metro').move(50, 90);
metro1.inlets['period'].receive(2000);
metro2.inlets['period'].receive(3000);

var random1 = patch.addNode('util/random').move(170, 10);
random1.inlets['max'].receive(26);
var random2 = patch.addNode('util/random').move(170, 120);
random2.inlets['max'].receive(26);

var letter1 = patch.addNode('util/letter').move(300, 10);
var letter2 = patch.addNode('util/letter').move(300, 110);

metro1.outlets['bang'].connect(random1.inlets['bang']);
metro2.outlets['bang'].connect(random2.inlets['bang']);

random1.outlets['random'].connect(letter1.inlets['code']);
random2.outlets['random'].connect(letter2.inlets['code']);

Rpd.nodetype('user/maybe-flag', {
    title: 'May be a flag?',
    inlets: {
        'letterA': { type: 'core/any' },
        'letterB': { type: 'core/any' }
    },
    outlets: {
        'char': { type: 'core/any' },
        'code': { type: 'core/any' }
    },
    process: function(inlets) {
        if (!inlets.letterA || !inlets.letterB) return;
        return { 'code': String.fromCharCode(inlets.letterA.charCodeAt(0) - 32)
                       + String.fromCharCode(inlets.letterB.charCodeAt(0) - 32),
                 'char' : fromCodePoint(55356)
                        + fromCodePoint(inlets.letterA.charCodeAt(0) - 97 + 56806)
                        + fromCodePoint(55356)
                        + fromCodePoint(inlets.letterB.charCodeAt(0) - 97 + 56806) };
    }
});

Rpd.noderenderer('user/maybe-flag', 'svg', function() {
    var textElm;
    return {
        first: function(bodyElm) {
            textElm = d3.select(bodyElm).append('text')
                        .style('text-anchor', 'middle');
        },
        always: function(bodyElm, inlets, outlets) {
            if (!outlets) return;
            textElm.text(outlets.char + ' (' + outlets.code + ')');
        }
    }
});

var maybeFlag = patch.addNode('user/maybe-flag', 'Maybe<Flag>').move(430, 70);
letter1.outlets['letter'].connect(maybeFlag.inlets['letterA']);
letter2.outlets['letter'].connect(maybeFlag.inlets['letterB']);

var logNode = patch.addNode('util/log', {}, {
    'svg': {
        size: { width: 210, height: 30 }
    }
}).move(550, 70);
maybeFlag.outlets['char'].connect(logNode.inlets['what']);
```

## HTML5 Canvas and Custom Toolkit

The Scene consists of seven (by default) shifted and colored squares is loaded and attached to HTML5 Canvas. It has a configuration object where values could be changed by user. Since it uses `requestAnimationFrame` for animation, on every frame it checks this configuration and redraws itself according to values specified there. So when values in this object are changed, Scene is immediately updated.

The Networks of given Nodes allows you to change configuration values. For example, by tuning the top left Knob, you may change the value of Red component of the starting color. By tuning the one in the middle, you may change the Y-coordinate of the shift between every two squares. You may take one of the free unconnected Knobs and connect it to Count Inlet of the Scene node to control the number of squares.

But if you connect the `result` Outlet of the Node named `%` to one or several of the `color` Node Inlets, you may interact with the Scene in even more direct way. `mouse` Node collects all of the movements of your mouse and converts them to the pair of X and Y coordinates. Then the mentioned `%` Node divides the value of X coordinate by modulus of 256, so the result is always between 0 and 256 (so, 0 <= result < 256) and always matches the amount `color` node wants as the input.

<div id="example-three"></div>

```javascript
/* ============== Coordinates Channel Type ============== */

Rpd.channeltype('my/coords', {
  show: function(val) {
    // nicely show a received pair of coordinates, floored to an integer
    return '<' + Math.floor(val.x) + ':' + Math.floor(val.y) + '>';
  }
});

/* ============== Coordinates Node Type ============== */

Rpd.nodetype('my/coords', {
  inlets: {
    x: { type: 'util/number', default: 0 },
    y: { type: 'util/number', default: 0 }
  },
  outlets: {
    out: { type: 'my/coords' }
  },
  // joins received `x` and `y` into one object
  process: function(inlets) {
    return { out: { x: inlets.x, y: inlets.y } };
  }
});

//* ============== Angle (radians) Channel Type ============== */

Rpd.channeltype('my/angle', {
  allow: [ 'util/number '], // outlets of `util/number` type are allowed to be
                            // connected to inlets of `my/angle` type
  accept: function(v) { return (v >= 0) && (v <= 360); },
  show: function(v) { return v + '˚'; }
});

/* ============== Canvas-driven Scene Node Type ============== */

var defaultConfig = {
  count: 7,
  from: { r: 0, g: 0, b: 0 },
  to: { r: 255, g: 0, b: 0 },
  shift: { x: 25, y: 0 },
  rotate: 15
};

Rpd.nodetype('my/scene', {
  inlets: {
    from: { type: 'util/color', 'default': defaultConfig.from },
    to: { type: 'util/color', 'default': defaultConfig.to },
    count: { type: 'util/number', 'default': defaultConfig.count,
             adapt: function(v) { return Math.floor(v); } },
    shift: { type: 'my/coords', 'default': defaultConfig.shift },
    rotate: { type: 'my/angle', 'default': defaultConfig.rotate },
  },
  process: function() {}
});

/* ============== Renderer for Canvas-driven Scene ============== */

var SVG_XMLNS = 'http://www.w3.org/2000/svg';

function lerp(v1, v2, pos) {
  return (v1 + ((v2 - v1) * pos));
}

Rpd.noderenderer('my/scene', 'svg', function() {
  var width = 100, height = 100;

  var context;
  var particles = [];
  var lastCount = 0;
  var config = defaultConfig;

  // function to render current state of the scene using requestAnimationFrame
  function draw() {
    if (context) {
      context.save();
      context.fillStyle = '#fff';
      context.fillRect(0, 0, width, height);
      context.fillStyle = '#000';
      particles.forEach(function(particle, i) {
        context.fillStyle = 'rgb(' +
          Math.floor(lerp(config.from.r, config.to.r,
                          1 / (particles.length - 1) * i)) + ',' +
          Math.floor(lerp(config.from.g, config.to.g,
                          1 / (particles.length - 1) * i)) + ',' +
          Math.floor(lerp(config.from.b, config.to.b,
                          1 / (particles.length - 1) * i)) + ')';
        context.fillRect(0, 0, 15, 15);
        context.translate(config.shift.x, config.shift.y);
        context.rotate(config.rotate * Math.PI / 180);
      });
      context.restore();
    }
    requestAnimationFrame(draw);
  }
  requestAnimationFrame(draw);

  // return actual renderer definition
  return {
    size: { width: width + 10, height: height + 10 },
    pivot: { x: 0, y: 0 },

    // on creation, add canvas to the node body
    first: function(bodyElm) {
      var group = document.createElementNS(SVG_XMLNS, 'g');
      group.setAttributeNS(null, 'transform', 'translate(5, 5)');
      var foreign = document.createElementNS(SVG_XMLNS, 'foreignObject');
      canvas = document.createElement('canvas');
      canvas.setAttributeNS(null, 'width', width + 'px');
      canvas.setAttributeNS(null, 'height', height + 'px');
      canvas.style.position = 'fixed';
      foreign.appendChild(canvas);
      group.appendChild(foreign);
      bodyElm.appendChild(group);

      context = canvas.getContext('2d');
    },

    // update config values using values from inlets
    always: function(bodyElm, inlets) {
      if (!isNaN(inlets.count) && (inlets.count != lastCount)) {
        particles = [];
        for (var i = 0; i < inlets.count; i++) {
          particles.push({});
        }
        lastCount = inlets.count;
      }
      if (inlets.from) config.from = inlets.from;
      if (inlets.to) config.to = inlets.to;
      if (inlets.shift) config.shift = inlets.shift;
      if (!isNaN(inlets.rotate)) config.rotate = inlets.rotate;
    }

  };
});

/* ============== Patch Structure ============== */

Rpd.renderNext('svg', document.getElementById('example-three'),
                      { style: 'compact-v' });

var patch = Rpd.addPatch('Generate Canvas Shapes').resizeCanvas(800, 205);

var scene = patch.addNode('my/scene').move(570, 5);
var color1 = patch.addNode('util/color').move(120, 5);
var color2 = patch.addNode('util/color').move(100, 80);
var coords = patch.addNode('my/coords').move(305, 90);
var knob1 = patch.addNode('util/knob').move(25, 5);
var knob2 = patch.addNode('util/knob').move(490, 110);
var knob3 = patch.addNode('util/knob').move(210, 105);
var knob4 = patch.addNode('util/knob').move(400, 110);
var mouse = patch.addNode('util/mouse-pos').move(0, 70);
var modulus = patch.addNode('util/mod').move(20, 150);
var comment = patch.addNode('util/comment').move(80, 100);

knob1.inlets['max'].receive(256);
knob2.inlets['max'].receive(180);
knob4.inlets['max'].receive(15);
coords.inlets['x'].receive(25);
modulus.inlets['b'].receive(256);
comment.inlets['text'].receive('Try to connect "%" node output' +
    + ' to inlet of "my/coords" node or one of the "color" nodes');

knob1.outlets['number'].connect(color1.inlets['r']);
knob3.outlets['number'].connect(coords.inlets['y']);
color1.outlets['color'].connect(scene.inlets['from']);
color2.outlets['color'].connect(scene.inlets['to']);
coords.outlets['out'].connect(scene.inlets['shift']);
mouse.outlets['x'].connect(modulus.inlets['a']);
```
