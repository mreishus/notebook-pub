# Mobile-first Web Animations in React

Talk by Alex Holacheck at React Conf 2019

## Definition of Mobile-first Animations

1. Takes inspiration from design patterns found in native apps, especially
   thoughtful use of gestures
2. Developed on a mobile device, intentionally designed to work in the context
   of a mobile browser rather than a desktop on
3. Has great performance, even on mid-range devices

## Horizontal Snapping Scroll

Tabs, cards, carosels..

```css
.tab-panels-container {
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  overflow-x: scroll;
  width: 100vw;
  display: flex;
}

.tab-panel {
  scroll-snap-align: start;
  /* stop: only supported in chrome */
  scroll-snap-stop: always;
}
```

## Swipe to dismiss

Swipe a panel down. Swipe an email right. Lots of different forms, but it's
everywhere.

She tried to implement from scratch 3 different times.

### First one - Lessons Learned

Worked only in Chrome Dev tools, but it was horrible on a Nokia 6 (\$130).
Jank, lag, intereference from default browser behaviors, slugging response to
touch, non-intuitive physics.

**Use a mobile device as your main dev UI.** Browse your app using the phone.
You can also do USB bugging.

## Implementation Options

- framer motion
- react spring + react use gesture
- react-swipeable + AnimeJS
- Hammer.js

## Spring / Use Gesture example

```javascript
import {animated, useSpring} from "react-spring";
import {useDrag} from "react-use-gesture";

const [ {y}, set] = useSpring( () => ({ y: 0 }))

const bind = useDrag(({ last, movement: [, movementY], memo = y.value }), {
  if (last) {
    const notificationClosed = movementY > 50;
    return set({
        y: notificationClosed ? 100 : 0,
        onRest: notificationClosed && removeNotification
    })
  }
  set({
    y: clamp(0, 100, memo + movementY)
  })

  return memo;
})

<StyledNotification
as={animated.div}
onTouchStart={bind().onTouchStart}
style={{
  opacity: y.interpolate( [0, 100], [1, 0] ),
  transform: y.interpolate(y => `translateY(${y}px)`)
}}
/>

```

## Scroll

1. Needs an Immediate Response, or feels super disconnected
2. Scroll Decay can help us decide how our UIs should respond

```css
-webkit-overflow-scrolling: touch;
```

Gives momentum scrolling on an overflowed container. Momentum scrolling uses decay rather than a spring or easing. Ios has two decay rates fast (0.99) and normal (0.998).

Swipe right on email example. Three options: 1. Light swipe: cancel option. 2. Medium swipe: Show archive/delete buttons. 3. Hard swipe: Delete directly.

```javascript
// Apple "Designing Fluid Interfaces"
const projection = startVelocity => (startVelocity * 0.998) / (1 - 0.998);

// usage
const velocityY = 1.2; // px/ms
const projectedEndpoint = projection(velocityY); // 679px
```

3. Rubberbanding

Rather than having a hard endpoint, there's an increasing friction. Even if
you're at the top of the list you can pull down and watch it "bounce".

```javascript
const rubberBand = (distance, dimension, constant = 0.55) =>
  (distance * dimension * constant) / (dimension + constant * distance);

const rubberBandClamp = (min, max, delta, constant) =>
  delta < min
    ? -rubberband(min - delta, max - min, constant) + min
    : delta > max
    ? rubberBand(delta - max, max - min, constant) + max
    : delta;
```

## Gestures and Springs

Springs are bouncy. Apple suggests tap = no bounce, swipe = a little bounce.

Springs are interruptable. Mobile animations should be interruptable.

Springs have velocity. The speed of the user's gesture should be matched by the
speed of the element moving.

## Animation Best Practices Recap

Use momentum scrolling as a reference point for the behaviors of gesture driven
animations.

Animations and gestures should be interruptable and redirectable at any point,
which is facilitated by using springs for animation.

Spring animations initiated after a user finished a gesture should accurately
reflect the velocity of the initiating gesture.

## The Mobile Browser

The mobile browser already handles a lot of gestures. Be careful about interference.

Technique 1: Hysteresis: Intentional lag between a user input and the moment
a system decides what to do with that input. (Intentional lag).

W/o it, scrolling up and down an email list might trigger sideways gestures
because you're not scrolling at a perfect 90 degree angle.

```javascript
const isIntentionalXAxisGesture = (movementX, movementY, threshold = 15) => {
  const isOverThreshold = Math.abs(movementX) > threshold;
  const isHorizontal = Math.abs(movementX) >= Math.abs(movementY);
  return isOverThreshold && isHorizontal;
};
```

Technique 2: Touch Cancellation

event.preventDefault()? You shouldn't use it for touch listeners, since you'll
need to add a non-passive event listener, which will kill scroll performance.

css property: `touch-action: all` (default). `none` (disable all gestures) `pan-x` (disable y-axis gestures) `pan-y` (disable x-axis gestures).

## Mobile Browser Recap

1. Use hysteresis to make sure users won't trigger animations unintentionally.
2. Use touch-action css property to disable default browser behaviors w/ precision.
3. Mobile browsers have a lot of differences in what they support.

## Performance

Use only transform and opacity: best gpu accelerated CSS properties.
`will-change: transform, opacity`.

Simpler is always more performant. What's the min you can do?

Keep animation start times under 100ms. (Use chrome performance profile tools - React.memo one possible solution)

## Resources

[Slides](https://mobile-first-animation.netlify.com)

[Code](https://github.com/aholachek/mobile-first-animation)

```
Created:       Fri 25 Oct 2019 07:16:17 PM CDT
Last Modified: Fri 25 Oct 2019 07:16:48 PM CDT
```
