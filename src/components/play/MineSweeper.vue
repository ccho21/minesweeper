<template>
    <div class="container mt-5 d-flex flex-column">

        <table class="minesweeper" oncontextmenu="return false">
            <caption class="py-0">
                <div class="heading">
                    <div class="bg-dark">
                        <div class="bg-dark">
                            <div class="digital-counter">{{flagNum}}</div>
                        </div>
                    </div>

                    <div class="smiley-box">
                        <a href="#" @click.prevent="startGame()">
                            <img v-bind:src="imgLink" height="25" width="25" class=""/>
                        </a>
                    </div>
                    <div class="bg-dark">
                        <div class="digital-counter">{{timer.count}}</div>
                    </div>
                </div>
            </caption>
            <tr v-for="(row, r) in tiles">
                <td v-for="(tile , t) in row"
                    @mouseup.prevent="ClickUp(r, t, $event)"
                    @mousedown.prevent="ClickDown(r, t, $event)"
                    @mouseover.prevent="clickHover(r, t, $event)"
                    @mouseleave.prevent="clickLeave(r, t, $event)"
                    :class="tile.classes">
                </td>
                <!-- @click.left.prevent="clickTile(r, t)"
                                   @contextmenu.right.prevent="flagTile(r, t, $event)"-->
                <!--@click.right.exact.prevent="flagTile(r, t, $event)"-->

            </tr>
        </table>
    </div>
</template>

<script>
    export default {
        name: "mindsweeper",
        data: function () {
            return {
                rows: 15,
                columns: 30,
                tiles: [],
                totalMine: 0,
                flagNum: 0,
                timer: {
                    time: false,
                    count: 0,
                    end: 0
                },
                result: false,
                imgLink: require('../../assets/img/smiley-normal.png'),
                leftToggle: false,
                rightToggle: false,
            }
        },
        created: function () {
            this.startGame();
            setInterval(this.timeCounter, 1000);
        },
        methods: {

            timeStop() {
                this.timer.time = false;
                this.timer.end = this.timer.count;
            },
            init() {
                this.totalMine = 0;
                this.timer.count = 0;
                this.timer.time = true;
                this.imgLink = require('../../assets/img/smiley-normal.png');
                this.leftToggle = false;
                this.rightToggle = false;
            },
            gameEnd(result) {
                if (result) {
                    this.imgLink = require('../../assets/img/smiley-win.png');
                    this.timeStop();
                }
                else {
                    this.imgLink = require('../../assets/img/smiley-dead.png');
                    this.timeStop();
                }
            },
            timeCounter() {
                if (this.timer.time) {
                    this.timer.count += 1;
                }
            },
            startGame(e) {
                this.init();
                let mineNum = 0;
                this.tiles = Array(this.rows).fill(0).map((_, row) => {
                    return Array(this.columns).fill(0).map((_, col) => {
                        let mine = Math.floor(Math.random() * 10) > 8.9999;
                        if (mine) {
                            this.totalMine += 1;
                        }
                        return {
                            row, col,
                            mined: mine,
                            classes: ['unopened']
                        }
                    });
                });
                // copy the total number of mine to flag number.
                // it will be decreased by 1 when flag event is triggered no matter what there is mine under the selected tile.
                this.flagNum = this.totalMine;
            },
            clickTile(row, column, unveiling_all) {
                let tile = this.tiles[row][column];
                // if there is no more tile to open;
                if (this.totalMine === 0) {
                    if (!unveiling_all) {
                        this.openAll();
                    }
                    this.gameEnd(true);
                }
                //find mine
                // console.log(tile.mined);
                // console.log(tile.col + "real col");
                // console.log(tile.row + "real row");
                if (tile.mined) {
                    tile.classes = ['mine'];
                    this.openAll();
                    this.gameEnd(false);
                }
                else {
                    //open tile;
                    if (this.checkValid(tile)) {
                        this.openTile(tile);
                    }
                }
            },
            openTile(t, unveiling_all) {
                let tile = t;
                // open the tile first because there is no mine.

                // Mine Open
                if (tile.mined) {
                    tile.classes = ['mine'];
                }
                else {
                    let neighbours = this.getNeighbours(tile);
                    let mineNum = this.countNeighbourMine(neighbours);
                    //OPEN neighbours when there is no mine around
                    if (mineNum > 0) {
                        tile.classes = ['mine-neighbour-' + mineNum];
                    }
                    else {
                        if (!unveiling_all) {
                            neighbours.forEach((t, i) => {
                                // if (this.checkValid(t)) {
                                t.classes = ['opened'];
                                this.openTile(t, false);
                                // }
                            });
                        }
                    }
                }
            },
            flagTile(row, col, e) {
                if (row !== '' && col !== '') {
                    let tile = this.tiles[row][col];
                    if (!this.checkValid(tile)) {
                        return;
                    }
                    if (tile.classes.indexOf('unopened') >= 0) {
                        if (tile.mined) {
                            this.totalMine -= 1;
                        }
                        tile.classes = ['flagged'];
                        this.flagNum -= 1;
                    }
                    else {
                        tile.classes = ['unopened'];
                        this.flagNum += 1;
                    }
                }
            },
            getNeighbours(t) {
                let neighbours = [];
                let surroundings = [[-1, -1], [-1, 0], [-1, 1],
                    [0, -1], [0, 1],
                    [1, -1], [1, 0], [1, 1]];
                surroundings.forEach((p) => {
                    let x = t.row + p[0];
                    let y = t.col + p[1];
                    if (this.checkLength(x, y)) {
                        let tile = this.tiles[x][y];
                        if (this.checkValid(tile)) {
                            neighbours.push(this.tiles[x][y]);
                        }
                    }
                });
                return neighbours;
            },
            openAll() {
                this.tiles.forEach((row, r) => {
                    row.forEach((tile, c) => {
                        let t = this.tiles[r][c];
                        this.openTile(t, true);
                    });
                })
            },
            countNeighbourMine(neighbour) {
                return neighbour.filter((cur) => {
                        return cur.mined;
                }).length;
            },
            checkLength(row, col) {
                return ((row >= 0 && row < this.tiles.length) && (col >= 0 && col < this.tiles[0].length))
            },
            checkValid(tile) {
                let reg = /mine-neighbour-[1-7]/g;
                return !(
                    (tile.classes[0] === 'opened') ||
                    (tile.classes[0].match(reg)) ||
                    (tile.classes[0] === 'mine')
                );
            },
            // MOUSE ACTIONS
            ClickDown(row, col, e) {
                let tile = this.tiles[row][col];
                if (e.which === 1) {
                    this.leftToggle = true;
                    // console.log('left down');
                    if (tile.classes[0] === 'unopened') {
                        tile.classes = ['hovered'];
                    }
                }
                if (e.which === 3) {
                    this.flagTile(row, col);
                }
                if (this.leftToggle && e.which === 3) {
                    this.rightToggle = true;
                    if (this.checkValid(tile)) {
                        tile.classes = ['hovered'];
                    }
                    this.groupHover(tile);
                }
            },
            ClickUp(row, col, e) {
                let tile = this.tiles[row][col];
                if (this.leftToggle && !this.rightToggle) {
                    let tile = this.tiles[row][col];
                    this.leftToggle = false;
                    this.clickTile(row, col);
                    // console.log('left up');
                }
                if (this.rightToggle) {
                    this.rightToggle = false;
                    this.leftToggle = false;
                    // console.log('right up');

                    let group = this.getNeighbours(tile);
                    let mineNum = this.countNeighbourMine(group);

                    let flagNum = group.filter((cur)=>{
                        return cur.classes[0] === 'flagged';
                    }).length;
                    console.log('mine Num' + mineNum);
                    console.log('flag num ' + flagNum);
                    if (mineNum - flagNum > 0) {
                        if (tile.classes[0] === 'hovered') {
                            tile.classes = ['unopened'];
                        }
                        this.groupLeave(tile);
                    }
                    else {
                        // open tiles
                        console.log(group);
                        group.map((cur, i) => {
                            if (cur.classes[0] === 'flagged') {
                                console.log(cur);
                            } else {
                                console.log('click Tile Start!!');
                                console.log(cur);
                                this.clickTile(cur.row, cur.col);
                            }
                        });
                    }
                }
            },
            clickHover(row, col, e) {
                let tile = this.tiles[row][col];
                if (this.leftToggle && !this.rightToggle) {
                    if (tile.classes[0] === 'unopened') {
                        tile.classes = ['hovered'];
                        // console.log('left hovered');
                    }
                }
                if (this.leftToggle && this.rightToggle) {
                    if (tile.classes[0] === 'unopened') {
                        tile.classes = ['hovered'];
                        // console.log('left hovered');
                    }
                    this.groupHover(tile);

                }
            },
            clickLeave(row, col, e) {
                let tile = this.tiles[row][col];

                if (this.leftToggle && !this.rightToggle) {
                    if (tile.classes[0] === 'hovered') {
                        tile.classes = ['unopened'];
                    }
                }
                else if (this.leftToggle && this.rightToggle) {

                    if (tile.classes[0] === 'hovered') {
                        tile.classes = ['unopened'];
                    }
                    this.groupLeave(tile)
                }
            },
            //MOUSE ACTION END!!!
            groupHover(tile) {
                let group = this.getNeighbours(tile);
                group.map((cur, i) => {
                    if (!(cur.classes[0] === 'flagged')) {
                        return cur.classes = ['hovered'];
                    }
                });
            },
            groupLeave(tile) {
                let group = this.getNeighbours(tile);
                group.map((cur, i) => {
                    if (cur.classes[0] === 'hovered') {
                        return cur.classes = ['unopened'];
                    }
                });
            }
        }
    }
</script>

<style scoped>
    .heading {
        border: .5rem double #6e6e6e;
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 100%;
        background-color: #bbbbbb;
    }

    .smiley-box {
        border: .2rem outset #999;
        padding: .5rem;
        margin: .5rem;
    }

    .smiley-box:active {
        border: .1rem solid #999;
    }

    table.minesweeper {
        border-collapse: collapse;
        background-color: #bbbbbb;
        margin: auto;
    }

    table.minesweeper caption {
        caption-side: top;
    }

    table.minesweeper td {
        width: 2rem;
        height: 2rem;
        background-size: cover;
        border: .1rem solid #727272;
    }

    .digital-counter {
        display: inline-block;
        background: #000;
        color: #F00;
        font-size: 2rem;
        font-family: Courier;
        padding: 0 .5rem;
    }
</style>