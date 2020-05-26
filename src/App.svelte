<script>
  import { onMount } from "svelte";

  let dragging = null;
  let element;
  let mousePosition = { x: 0, y: 0 };

  let state = {
    items: [
      {
        id: 1,
        color: "red",
        left: 0,
        top: 0,
        width: 200,
        height: 100,
        points: [{ A: [0, 0], B: [200, 0], C: [0, 100], D: [200, 100] }]
      },
      {
        id: 2,
        color: "blue",
        left: 250,
        top: 250,
        width: 300,
        height: 100,
        points: [{ A: [250, 250], B: [550, 250], C: [250, 350], D: [550, 350] }]
      },
      {
        id: 3,
        color: "yellow",
        left: 500,
        top: 500,
        width: 100,
        height: 200,
        points: [{ A: [500, 500], B: [600, 500], C: [500, 700], D: [600, 700] }]
      }
    ]
  };

  onMount(() => {
    window.addEventListener("mouseup", handleMouseUp);
    window.addEventListener("mousemove", handleMouseMove);
    //setPoints();
  });

  function handleMouseUp(e) {
    getPoints();
    dragging = null;
  }

  function handleMouseDown(e, index) {
    const item = state.items[index];

    state.items.splice(index, 1);
    state.items = [...state.items, item];

    dragging = {
      x: e.clientX,
      y: e.clientY,
      left: item.left,
      top: item.top,
      index: state.items.length - 1
    };
  }

  function checkTopBottom(draggingIndex, index) {
    let returnValue = false;
    const myTop = state.items[draggingIndex].top;
    const itsTop = state.items[index].top;
    const difference = itsTop - myTop;

    if (
      -1 * state.items[index].height < difference &&
      difference < state.items[index].height
    ) {
      returnValue = true;
    }
    return returnValue;
  }

  function checkLeftRight(draggingIndex, index) {
    let returnValue = false;
    const myLeft = state.items[draggingIndex].left;
    const itsLeft = state.items[index].left;
    const difference = itsLeft - myLeft;
    if (
      -1 * state.items[index].width < difference &&
      difference < state.items[index].width
    ) {
      returnValue = true;
    }
    return returnValue;
  }

  function getSnapped(draggingIndex, i) {
    const myLeft = state.items[dragging.index].left;
    const itsRight = state.items[i].left + state.items[i].width;
    const myRight =
      state.items[dragging.index].left + state.items[dragging.index].width;
    const itsLeft = state.items[i].left;
    const myTop = state.items[dragging.index].top;
    const itsTop = state.items[i].top;
    const myBottom =
      state.items[dragging.index].top + state.items[dragging.index].height;
    const itsBottom = state.items[i].top + state.items[i].height;

    let isTopBottominRange = checkLeftRight(draggingIndex, i);
    let isLeftRightinRange = checkTopBottom(draggingIndex, i);

    if (
      myLeft > itsRight - 20 &&
      myLeft < itsRight + 20 &&
      isLeftRightinRange
    ) {
      state.items[dragging.index].left = itsRight;
    }
    if (
      myRight > itsLeft - 20 &&
      myRight < itsLeft + 20 &&
      isLeftRightinRange
    ) {
      state.items[dragging.index].left =
        itsLeft - state.items[dragging.index].width;
    }
    if (
      myTop > itsBottom - 20 &&
      myTop < itsBottom + 20 &&
      isTopBottominRange
    ) {
      state.items[dragging.index].top = itsBottom;
    }
    if (
      myBottom > itsTop - 20 &&
      myBottom < itsTop + 20 &&
      isTopBottominRange
    ) {
      state.items[dragging.index].top =
        itsTop - state.items[dragging.index].height;
    }
    if (myLeft > itsLeft - 20 && myLeft < itsLeft + 20 && isLeftRightinRange) {
      state.items[dragging.index].left = itsLeft;
    }
    if (
      myRight > itsRight - 20 &&
      myRight < itsRight + 20 &&
      isLeftRightinRange
    ) {
      state.items[dragging.index].left = itsLeft;
    }
    if (myTop > itsTop - 20 && myTop < itsTop + 20 && isTopBottominRange) {
      state.items[dragging.index].top = itsTop;
    }
    if (
      myBottom > itsBottom - 20 &&
      myBottom < itsBottom + 20 &&
      isTopBottominRange
    ) {
      state.items[dragging.index].top = itsTop;
    }
  }

  function getPoints() {
    for (let i = 0; i < state.items.length; i++) {
      const left = state.items[i].left;
      const top = state.items[i].top;
      const right = state.items[i].left + state.items[i].width;
      const bottom = state.items[i].top + state.items[i].height;

      state.items[i].points[0].A = [left, top];
      state.items[i].points[0].B = [right, top];
      state.items[i].points[0].C = [left, bottom];
      state.items[i].points[0].D = [right, bottom];
    }
    checkIntersection(state.items[2].points);
  }

  let target = {
    A: { X: 0, Y: 0 },
    B: { X: 0, Y: 0 },
    C: { X: 0, Y: 0 },
    D: { X: 0, Y: 0 }
  };

  function checkIntersection(draggingRectangle) {
    target.A.X = state.items[1].points[0].A[0];
    target.A.Y = state.items[1].points[0].A[1];

    target.B.X = state.items[1].points[0].B[0];
    target.B.Y = state.items[1].points[0].B[1];

    target.C.X = state.items[1].points[0].C[0];
    target.C.Y = state.items[1].points[0].C[1];

    target.D.X = state.items[1].points[0].D[0];
    target.D.Y = state.items[1].points[0].D[1];

    if (
      (target.B.Y > draggingRectangle[0].B[1] &&
        target.B.Y < draggingRectangle[0].D[1]) ||
      (target.B.Y < draggingRectangle[0].A[1] &&
        target.D.Y > draggingRectangle[0].A[1])
    ) {
      if (
        target.B.X > draggingRectangle[0].A[0] &&
        target.B.X < draggingRectangle[0].B[0]
      ) {
        console.log("Intersecting from right!");
      }
    }

    if (
      (target.A.Y > draggingRectangle[0].A[1] &&
        target.A.Y < draggingRectangle[0].C[1]) ||
      (target.A.Y < draggingRectangle[0].A[1] &&
        target.C.Y > draggingRectangle[0].A[1])
    ) {
      if (
        target.A.X > draggingRectangle[0].A[0] &&
        target.A.X < draggingRectangle[0].B[0]
      ) {
        console.log("Intersecting from left!");
      }
    }

    if (
      target.B.Y > draggingRectangle[0].B[1] &&
      target.B.Y < draggingRectangle[0].D[1]
    )
      if (
        target.A.X < draggingRectangle[0].A[0] &&
        target.B.X > draggingRectangle[0].B[0]
      ) {
        console.log("Interecting from top!");
      }
    if (
      target.D.Y > draggingRectangle[0].B[1] &&
      target.D.Y < draggingRectangle[0].D[1]
    ) {
      if (
        target.A.X < draggingRectangle[0].A[0] &&
        target.B.X > draggingRectangle[0].B[0]
      ) {
        console.log("Interecting from bottom!");
      }
    }
  }
  function handleMouseMove(e) {
    if (dragging) {
      const deltaX = e.clientX - dragging.x;
      const deltaY = e.clientY - dragging.y;

      state.items[dragging.index].left = dragging.left + deltaX;
      state.items[dragging.index].top = dragging.top + deltaY;

      for (let i = 0; i < state.items.length; i++) {
        if (i === dragging.index) {
          continue;
        }
        getPoints();
        //getSnapped(dragging.index, i);  this is for snapping rectangles each other when they come close enough
      }
    }
  }
</script>

<style>
  div {
    border: 1px solid black;
    position: absolute;
    background-color: red;
    user-select: none;
  }
</style>

{#each state.items as item, index}
  <div
    style="left:{item.left}px; background-color:{item.color};top:{item.top}px;
    width:{item.width}px; height:{item.height}px"
    on:mousedown={e => handleMouseDown(e, index)} />
{/each}
