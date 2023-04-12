<template>
  <div class="board">
    <div v-for="(row, r) in [ 8, 7, 6, 5, 4, 3, 2, 1 ]" class="row">
      <div v-for="(cell, c) in ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']"
           :class="{black: (r+c)%2===1, 'can-move-to': moveIsInMoveset(possible_moves_for_selected_piece, [r,c]) }"
           class="cell"
           @click="()=>cellClicked(r,c)">
        <div class="cell-content">
          <img v-if="getPieceAtPosition(r, c)" :src="getPieceImage(getPieceAtPosition(r, c))" alt="">
        </div>
      </div>
    </div>
  </div>
  <p class="error">{{ error_message }}</p>
  <p>It's {{ whites_turn ? "White" : "Black" }}'s turn</p>
  <p v-if="piece_to_move">{{ piece_to_move }} {{getPieceAtPosition(piece_to_move.r, piece_to_move.c)}} selected</p>
  <p v-else>No Piece selected</p>
</template>

<script>
export default {
  name: "Chessboard",
  methods: {
    cellClicked(r, c) {
      console.log("cell clicked", r, c);
      if (!this.piece_to_move) {
        if (!this.getPieceAtPosition(r, c)) {
          return;
        }
        this.piece_to_move = {r, c};
        this.getMoveList(r, c, this.getPieceAtPosition(r, c));
      } else {
        this.movePiece(this.piece_to_move.r, this.piece_to_move.c, r, c);
        this.piece_to_move = null;
      }
    },
    getPieceAtPosition(row, col) {
      return this.pieces[row]?.[col];
    },
    coordinateToCellName(row, col) {
      return [row + 1, String.fromCharCode(97 + col)];
    },
    getPieceImage(piece) {
      const file_map = {
        "rook_w": "Chess_rlt60",
        "knight_w": "Chess_nlt60",
        "bishop_w": "Chess_blt60",
        "queen_w": "Chess_qlt60",
        "king_w": "Chess_klt60",
        "pawn_w": "Chess_plt60",
        "rook_b": "Chess_rdt60",
        "knight_b": "Chess_ndt60",
        "bishop_b": "Chess_bdt60",
        "queen_b": "Chess_qdt60",
        "king_b": "Chess_kdt60",
        "pawn_b": "Chess_pdt60"
      }


      return `/images/pieces/${file_map[piece]}.png`;
    },
    validateMove(r_start, c_start, r_target, c_target, piece_type) {
      // Who is moving next?
      const is_white = piece_type.includes("_w");
      if (this.whites_turn !== is_white) {
        this.error_message = "Not your turn";

        return false;
      }

      const move_list = this.getMoveList(r_start, c_start, piece_type);
      if (!move_list.some(move => move[0] === r_target && move[1] === c_target)) {
        this.error_message = "Invalid move for this piece";
        return false;
      }


      this.error_message = "";
      return true;
    },
    moveIsInMoveset(moveSet, move) {
      return moveSet.some(m => m[0] === move[0] && m[1] === move[1]);
    },
    getMoveList(r, c, piece_type) {
      const is_white = piece_type.includes("_w");

      switch (piece_type) {
        case "rook_w":
        case "rook_b":
          return this.getRookMoveList(r, c, is_white);
        case "knight_w":
        case "knight_b":
          return this.getKnightMoveList(r, c, is_white);
        case "bishop_w":
        case "bishop_b":
          return this.getBishopMoveList(r, c, is_white);
        case "queen_w":
        case "queen_b":
          return this.getQueenMoveList(r, c, is_white);
        case "king_w":
        case "king_b":
          return this.getKingMoveList(r, c, is_white);
        case "pawn_w":
        case "pawn_b":
          return this.getPawnMoveList(r, c, is_white);
      }
    },

    getPawnMoveList(r, c, is_white) {
      const move_direction = is_white ? 1 : -1;

      console.log(this.getPieceAtPosition(r + 1 * move_direction, c), this.getPieceAtPosition(r + 1 * move_direction, c + 1), this.getPieceAtPosition(r + 1 * move_direction, c - 1))

      const move_list = [];
      if (this.getPieceAtPosition(r + 1 * move_direction, c) === undefined) {
        move_list.push([r + 1 * move_direction, c]);
      }
      if (this.getPieceAtPosition(r + 1 * move_direction, c + 1) !== undefined) {
        if (this.getPieceAtPosition(r + 1 * move_direction, c + 1).includes("_w") !== is_white) {
          move_list.push([r + 1 * move_direction, c + 1]);
        }
      }
      if (this.getPieceAtPosition(r + 1 * move_direction, c - 1) !== undefined) {
        if (this.getPieceAtPosition(r + 1 * move_direction, c - 1).includes("_w") !== is_white) {
          move_list.push([r + 1 * move_direction, c - 1]);
        }
      }

      if (r === 1 && is_white || r === 6 && !is_white) {
        if (this.getPieceAtPosition(r + 2 * move_direction, c) === undefined) {
          move_list.push([r + 2 * move_direction, c]);
        }
      }

      console.log(move_list);
      this.possible_moves_for_selected_piece = move_list;
      return move_list;
    },

    movePiece(r_start, c_start, r_target, c_target) {
      const piece_type = this.getPieceAtPosition(r_start, c_start);

      if (!this.validateMove(r_start, c_start, r_target, c_target, piece_type)) {
        this.piece_to_move = null;
        return;
      }


      if (!this.pieces[r_target]) {
        this.pieces[r_target] = {};
      }
      this.pieces[r_target][c_target] = piece_type;
      this.pieces[r_start][c_start] = undefined;

      this.whites_turn = !this.whites_turn;
      if (this.check_for_checkmate()) {
        this.error_message = "Checkmate!"
      }
      if (this.check_for_pat()) {
        this.error_message = "Pat!"
      }

      this.possible_moves_for_selected_piece = [];

    },
    check_for_checkmate() {
      return false;
    },
    check_for_pat() {
      return false;
    }
  },
  data() {
    return {
      possible_moves_for_selected_piece: [],
      whites_turn: true,
      error_message: "",
      piece_to_move: null,
      pieces: {
        0: {
          0: "rook_w",
          1: "knight_w",
          2: "bishop_w",
          3: "queen_w",
          4: "king_w",
          5: "bishop_w",
          6: "knight_w",
          7: "rook_w"
        },
        1: {
          0: "pawn_w",
          1: "pawn_w",
          2: "pawn_w",
          3: "pawn_w",
          4: "pawn_w",
          5: "pawn_w",
          6: "pawn_w",
          7: "pawn_w"
        },
        6: {
          0: "pawn_b",
          1: "pawn_b",
          2: "pawn_b",
          3: "pawn_b",
          4: "pawn_b",
          5: "pawn_b",
          6: "pawn_b",
          7: "pawn_b"
        },
        7: {
          0: "rook_b",
          1: "knight_b",
          2: "bishop_b",
          3: "queen_b",
          4: "king_b",
          5: "bishop_b",
          6: "knight_b",
          7: "rook_b"
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
$board-size: 500px;
$cell-size: $board-size / 8;
$dark-color: #222;
$light-color: #fff;

.board {
  display: flex;
  flex-direction: column;
  width: $board-size;
  height: $board-size;

  .row {
    display: flex;

    .cell {
      width: $cell-size;
      height: $cell-size;
      outline: 1px solid $dark-color;

      &.black {
        background-color: $dark-color;
        color: $light-color;
      }

      &.can-move-to {
        background-color: green;
      }

      .cell-content {
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;

        img {
          width: 100%;
          height: 100%;
        }
      }
    }
  }
}

.error {
  color: red;
}

</style>