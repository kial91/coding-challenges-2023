
<script lang="ts">
	import { onMount } from "svelte";

    enum Direction {
        North,
        East,
        South,
        West,
    };
    enum TileState {
        Initial = "-",
        Invert = "X"
    };

    // Initialise variables
    let rowLength = 8;
    let columnLength = 7;
    let grid: string[][] = [];
    for (let row = 0; row < rowLength; row++) {
        grid.push(Array(columnLength).fill(TileState.Initial));
    }
    const ant = {
        pos: [2, 2],
        direction: Direction.North
    };

    // Ant PNG
    const antImage = "/ant.png";

    // Helper functions
    // Tile state
    function setTileState(row: number, col: number, state: TileState) {
        grid[row][col] = state;
    }
    function getTileState(row: number, col: number) {
        return grid[row][col];
    }

    // Ant positioning
    function updateAntPos() {

        switch(ant.direction) {
            case Direction.North: {
                ant.pos = [ant.pos[0] - 1, ant.pos[1]];
                break;
            }
            case Direction.East: {
                ant.pos = [ant.pos[0], ant.pos[1] + 1];
                break;
            }
            case Direction.South: {
                ant.pos = [ant.pos[0] + 1, ant.pos[1]];
                break;
            }
            case Direction.West: {
                ant.pos = [ant.pos[0], ant.pos[1] - 1];
                break;
            }
        }
    }

    // Grid
    function checkIfOutOfBoundary() {
        
        switch(ant.direction) {
            case Direction.North: {
                return ant.pos[0] < 0;
            }
            case Direction.East: {
                return ant.pos[1] > (columnLength - 1);
            }
            case Direction.South: {
                return ant.pos[0] > (rowLength - 1);
            }
            case Direction.West: {
                return ant.pos[1] < 0;
            }
        }
    }

    function expandGrid() {

        // East
        for (const row of grid) {
            row.push(TileState.Initial);
        }

        // West
        for (const row of grid) {
            row.unshift(TileState.Initial);
        }
        columnLength += 2;

        // North/South
        grid.unshift(Array(columnLength).fill(TileState.Initial));
        grid.push(Array(columnLength).fill(TileState.Initial));
        rowLength += 2;
    }

    // Movement
    function move() {

         // Get current tile state to determine next direction
        // Turn left if on white tile and right on black
        const tileState = getTileState(ant.pos[0], ant.pos[1]);
        switch(ant.direction) {
            case Direction.North: {
                ant.direction = tileState === TileState.Initial ? Direction.West : Direction.East;
                break;
            }
            case Direction.East: {
                ant.direction = tileState === TileState.Initial ? Direction.North : Direction.South;
                break;
            }
            case Direction.South: {
                ant.direction = tileState === TileState.Initial ? Direction.East : Direction.West;
                break;
            }
            case Direction.West: {
                ant.direction = tileState === TileState.Initial ? Direction.South : Direction.North;
                break;
            }
        }

        // Set new tile state
        setTileState(ant.pos[0], ant.pos[1], grid[ant.pos[0]][ant.pos[1]] === TileState.Initial ? TileState.Invert : TileState.Initial);

        // Update ant position
        updateAntPos();

        // Check if the ant is now out of the boundary
        // If it is we need to update the grid size
        if (checkIfOutOfBoundary()) {

            // Re-adjust ant pos
            ant.pos = [ant.pos[0] + 1, ant.pos[1] + 1];

            // Expand grid on all sides
            expandGrid();
        }

        // Force update grid
        grid = grid;

        // console.log(grid);
    }

    onMount(() => {
        let interval: NodeJS.Timer;

        // Start
        (window as any).startAnt = function () {
            interval = setInterval(() => {
                move();
            }, 300);
        };

        // Stop
        (window as any).stopAnt = function () {
            clearInterval(interval);
        };

        (window as any).startAnt();
    });
</script>

<div class="grid" style='--angle:{ant.direction === Direction.North ? 0 : ant.direction === Direction.East ? 90 :
                                  ant.direction === Direction.South ? 180 : 270}deg;'>
    {#each grid as row, rowPos}
        <div class="row">
            {#each row as tile, colPos}
                <div class={`tile ${tile === TileState.Invert ? "invert" : "initial" }`}>
                    <!-- {`(${rowPos}, ${colPos})`} -->
                    {#if ant.pos[0] === rowPos && ant.pos[1] === colPos}
                        <img class="ant" src={antImage} alt="ant"/>
                    {/if}
                </div>
            {/each}
        </div>
    {/each}   
</div>

<button on:click={move}>Move</button>

<style>
    .row {
        display: flex;
    }

    .tile {
        outline: 1px solid #000000;
        padding: 20px;
        height: 30px;
        width: 30px;
        overflow: hidden;
        position: relative;
    }

    .tile.initial {
        background-color: #FFFFFF;
    }

    .tile.invert {
        background-color: #CCCCCC;
    }

    .tile.hidden {
        visibility: hidden;
    }

    .ant {
        height: 40px;
        width: 40px;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        margin: auto;
        transform: rotate(var(--angle));
    }

    button {
        display: none;
    }
</style>