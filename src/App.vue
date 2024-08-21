<template>
    <div id="app">
        <h1>Крестики-Нолики</h1>
        <p>Счет игры X: {{ scoreX }} | O: {{ scoreO }}</p>
        <p>{{ status }}</p>
        <Board :board="board" @make-move="makeMove" ref="board" />
        <DifficultyLevel :difficulty="difficulty" @set-difficulty="setDifficulty" />
        <button @click="resetGame">Начать заново</button>
    </div>
</template>

<script>
import Board from './components/Board.vue';
import DifficultyLevel from './components/DifficultyLevel.vue';

export default {
    components: {
        Board,
        DifficultyLevel
    },
    data() {
        return {
            board: Array(9).fill(null),
            currentPlayer: 'X',
            winner: null,
            difficulty: 'medium',
            scoreX: 0,
            scoreO: 0
        };
    },
    computed: {
        status() {
            if (this.winner) {
                return `Игрок ${this.winner} выиграл!`;
            } else if (this.board.every(cell => cell !== null)) {
                return 'Ничья!';
            } else {
                return `Сейчас ходит игрок ${this.currentPlayer}`;
            }
        }
    },
    methods: {
        makeMove(index) {
            if (!this.board[index] && !this.winner) {
                this.board[index] = this.currentPlayer;
                this.checkWinner();
                if (!this.winner) {
                    this.currentPlayer = this.currentPlayer === 'X' ? 'O' : 'X';
                    if (this.currentPlayer === 'O') {
                        this.computerMove();
                    }
                }
            }
        },
        checkWinner() {
            const winningCombinations = [
                [0, 1, 2], [3, 4, 5], [6, 7, 8],
                [0, 3, 6], [1, 4, 7], [2, 5, 8],
                [0, 4, 8], [2, 4, 6]
            ];
            for (let combination of winningCombinations) {
                const [a, b, c] = combination;
                if (this.board[a] && this.board[a] === this.board[b] && this.board[a] === this.board[c]) {
                    this.winner = this.board[a];
                    if (this.winner === 'X') {
                        this.scoreX++;
                    } else if (this.winner === 'O') {
                        this.scoreO++;
                    }
                    this.$refs.board.$refs[`cell-${a}`][0].classList.add('glow');
                    this.$refs.board.$refs[`cell-${b}`][0].classList.add('glow');
                    this.$refs.board.$refs[`cell-${c}`][0].classList.add('glow');
                    return;
                }
            }
        },
        resetGame() {
            this.board = Array(9).fill(null);
            this.currentPlayer = 'X';
            this.winner = null;
            for (let i = 0; i < 9; i++) {
                this.$refs.board.$refs[`cell-${i}`][0].classList.remove('glow');
            }
        },
        setDifficulty(level) {
            this.difficulty = level;
        },
        computerMove() {
            if (this.difficulty === 'easy') {
                this.easyMove();
            } else if (this.difficulty === 'medium') {
                this.mediumMove();
            } else if (this.difficulty === 'hard') {
                this.hardMove();
            }
            this.checkWinner();
            this.currentPlayer = 'X';
        },
        easyMove() {
            let emptyCells = this.board.map((cell, index) => cell === null ? index : null).filter(index => index !== null);
            if (emptyCells.length > 0) {
                let randomIndex = emptyCells[Math.floor(Math.random() * emptyCells.length)];
                this.board[randomIndex] = 'O';
            }
        },
        mediumMove() {
            let move = this.findWinningMove('O');
            if (move === null) {
                move = this.findWinningMove('X');
            }
            if (move === null) {
                this.easyMove();
            } else {
                this.board[move] = 'O';
            }
        },
        hardMove() {
            let move;
            if (this.board[4] === null) {
                move = 4;
            } else {
                let corners = [0, 2, 6, 8];
                for (let corner of corners) {
                    if (this.board[corner] === null) {
                        move = corner;
                        break;
                    }
                }
            }
            if (move !== undefined) {
                this.board[move] = 'O';
            } else {
                let bestScore = -Infinity;
                for (let i = 0; i < this.board.length; i++) {
                    if (this.board[i] === null) {
                        this.board[i] = 'O';
                        let score = this.minimax(this.board, 0, -Infinity, Infinity, true);
                        this.board[i] = null;
                        if (score > bestScore) {
                            bestScore = score;
                            move = i;
                        }
                    }
                }
                this.board[move] = 'O';
            }
        },
        findWinningMove(player) {
            const winningCombinations = [
                [0, 1, 2], [3, 4, 5], [6, 7, 8],
                [0, 3, 6], [1, 4, 7], [2, 5, 8],
                [0, 4, 8], [2, 4, 6]
            ];
            for (let combination of winningCombinations) {
                const [a, b, c] = combination;
                if (this.board[a] === player && this.board[b] === player && this.board[c] === null) {
                    return c;
                }
                if (this.board[a] === player && this.board[b] === null && this.board[c] === player) {
                    return b;
                }
                if (this.board[a] === null && this.board[b] === player && this.board[c] === player) {
                    return a;
                }
            }
            return null;
        },
        minimax(board, depth, alpha, beta, isMaximizing) {
            let result = this.checkWinnerForMinimax(board);
            if (result !== null) {
                return result;
            }
            if (isMaximizing) {
                let bestScore = -Infinity;
                for (let i = 0; i < board.length; i++) {
                    if (board[i] === null) {
                        board[i] = 'O';
                        let score = this.minimax(board, depth + 1, alpha, beta, false);
                        board[i] = null;
                        bestScore = Math.max(score, bestScore);
                        alpha = Math.max(alpha, score);
                        if (beta <= alpha) {
                            break;
                        }
                    }
                }
                return bestScore;
            } else {
                let bestScore = Infinity;
                for (let i = 0; i < board.length; i++) {
                    if (board[i] === null) {
                        board[i] = 'X';
                        let score = this.minimax(board, depth + 1, alpha, beta, true);
                        board[i] = null;
                        bestScore = Math.min(score, bestScore);
                        beta = Math.min(beta, score);
                        if (beta <= alpha) {
                            break;
                        }
                    }
                }
                return bestScore;
            }
        },
        checkWinnerForMinimax(board) {
            const winningCombinations = [
                [0, 1, 2], [3, 4, 5], [6, 7, 8],
                [0, 3, 6], [1, 4, 7], [2, 5, 8],
                [0, 4, 8], [2, 4, 6]
            ];
            for (let combination of winningCombinations) {
                const [a, b, c] = combination;
                if (board[a] && board[a] === board[b] && board[a] === board[c]) {
                    if (board[a] === 'O') return 1;
                    if (board[a] === 'X') return -1;
                }
            }
            if (board.every(cell => cell !== null)) {
                return 0;
            }
            return null;
        }
    }
};
</script>

<style></style>