`timescale 1ns/1ps

// =============================
// VENDING MACHINE MODULE
// =============================
module vending(
    input clk,
    input reset,
    input coin5,
    input coin10,
    output reg product, 
    output reg change
);

    parameter S0  = 2'd0;
    parameter S5  = 2'd1;
    parameter S10 = 2'd2;
    parameter S15 = 2'd3;

    reg [1:0] state, next_state;

    // State register
    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= S0;
        else
            state <= next_state;
    end

    // Next-state & outputs (combinational)
    always @(*) begin
        next_state = state;
        product = 0;
        change = 0;

        case (state)
            S0: begin
                if (coin5)   next_state = S5;
                else if (coin10) next_state = S10;
            end

            S5: begin
                if (coin5)   next_state = S10;
                else if (coin10) next_state = S15;
            end

            S10: begin
                if (coin5) next_state = S15;
                else if (coin10) begin
                    next_state = S0;
                    product = 1;
                    change = 1; // extra ₹5
                end
            end

            S15: begin
                product = 1;
                next_state = S0;
            end

            default: next_state = S0;
        endcase
    end

endmodule

// =============================
// TESTBENCH (single-file for JDoodle)
// =============================
module vending_tb;

    reg clk;
    reg reset;
    reg coin5;
    reg coin10;
    wire product;
    wire change;

    // instantiate
    vending uut(
        .clk(clk), .reset(reset),
        .coin5(coin5), .coin10(coin10),
        .product(product), .change(change)
    );

    // Clock: initial + forever is more portable than 'always' at top level on some runners
    initial begin
        clk = 0;
        forever #5 clk = ~clk; // 10 ns period
    end

    // Observe signals on console
    initial begin
        $display("time\treset\tcoin5\tcoin10\tproduct\tchange");
        $monitor("%0t\t%b\t%b\t%b\t%b\t%b", $time, reset, coin5, coin10, product, change);
    end

    initial begin
        // init
        reset = 1; coin5 = 0; coin10 = 0;
        #12;          // give a bit of time
        reset = 0;

        // Insert 5 + 10 = 15 (should dispense)
        #8; coin5 = 1; #10; coin5 = 0;   // one 5-rupee
        #4; coin10 = 1; #10; coin10 = 0; // then 10-rupee

        // wait a bit
        #20;

        // Insert 10 + 10 = 20 (product + change)
        coin10 = 1; #10; coin10 = 0;
        #4; coin10 = 1; #10; coin10 = 0;

        #20;

        // Insert 5 + 5 + 5 = 15
        coin5 = 1; #10; coin5 = 0;
        #4; coin5 = 1; #10; coin5 = 0;
        #4; coin5 = 1; #10; coin5 = 0;

        #50 $finish;
    end

endmodule
