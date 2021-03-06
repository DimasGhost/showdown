<template>
    <div class="game-board-container">
        <div class="game-info">
            <div class="game-info-part x">
                <div class="marker">X</div> - {{match.participants[0].name}}
            </div>
            <div class="game-info-part zero">
                <div class="marker">O</div> - {{match.participants[1].name}}
            </div>
        </div>
        <div class="game-board">
            <div class="game-board-row" v-for="row in this.field">
                <div class="game-board-cell"
                     v-for="cell in row"
                     :class="cell === '' ? 'empty-cell' : (cell === 'X' ? 'x' : 'zero')">
                    {{cell}}
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
    @import url('https://fonts.googleapis.com/css?family=Permanent+Marker&display=swap');

    $cl-bg: #554e6e;
    $cl-zero: #ff6dbe;
    $cl-x: #4cdeff;

    .x {
        color: $cl-x;
    }

    .zero {
        color: $cl-zero;
    }

    .game-board-container {
        background: $cl-bg;
        display: flex;
        flex-direction: column;
        padding: 10px;
    }

    .game-info {
        margin-bottom: 10px;
        width: 400px;
        display: flex;
        justify-content: space-between;
    }

    .game-info-part {
        display: flex;
        align-items: center;
        margin-left: 10px;
        margin-right: 10px;
    }

    .marker {
        font-family: 'Permanent Marker', cursive;
    }

    .game-board {
        width: 400px;
        height: 400px;
        display: flex;
        flex-direction: column;
        border-top: 3px white solid;
        border-left: 3px white solid;
        background: $cl-bg;
        font-family: 'Permanent Marker', cursive;
    }

    .game-board-row {
        display: flex;
        width: 100%;
        height: 10%;
        border-bottom: 2px white solid;
    }

    .game-board-cell {
        width: 10%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        border-right: 3px white solid;
        user-select: none;
        font-size: 30px;
        transition: all .3s ease;
        overflow: hidden;
    }

    .empty-cell {
        font-size: 50px;
    }
</style>

<script>
    const X_LEN = 10;
    const Y_LEN = 10;

    export default {
        props: {
            cur_frame: { required: true },
            game_log: { required: true },
            match: { required: true }
        },
        data() {
            return {
                field: [],
                last_frame: -1,
            }
        },
        mounted() {
            for (let y = 0; y < Y_LEN; ++y) {
                let r = [];
                for (let x = 0; x < X_LEN; ++x) {
                    r.push('');
                }
                this.field.push(r);
            }
            this.redraw_field();
        },
        methods: {
            redraw_field() {
                if (this.last_frame < this.cur_frame) {
                    for (let i = this.last_frame + 1; i <= this.cur_frame; ++i) {
                        let move = this.game_log[i];
                        let x = move.move[0], y = move.move[1];
                        let id = move.bot_id;
                        let c = 'X';
                        if (id === this.match.participants[1].id)
                            c = 'O';
                        this.$set(this.field[y], x, c);
                    }
                } else {
                    for (let i = this.last_frame; i > this.cur_frame; --i) {
                        let move = this.game_log[i];
                        let x = move.move[0], y = move.move[1];
                        this.$set(this.field[y], x, '');
                    }
                }
                this.last_frame = this.cur_frame;
            }
        },
        watch: {
            cur_frame() {
                this.redraw_field();
            }
        }
    }
</script>
