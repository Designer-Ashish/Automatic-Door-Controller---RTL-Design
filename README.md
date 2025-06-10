# Automatic_Door_Controller-RTL_Design
# Specifications
The automatic door controller should:

1. Detect approaching persons using motion sensors

2. Open the door when a person is detected

3. Keep the door open as long as movement continues to be detected

4. Close the door after a timeout period when no more movement is detected

5. Include safety features to prevent closing when obstructed


                    +-------------------+
                    |  Motion Sensor    |
                    +---------+---------+
                              |
                    +---------v---------+
                    |   Sensor Interface |
                    +---------+---------+
                              |
                    +---------v---------+
                    |   Control Logic   |
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Timer/Counter    |
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Door Motor Driver|
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Obstruction      |
                    |  Sensor          |
                    +-------------------+

# RTL Implementation (Verilog)

module automatic_door_controller (
    input wire clk,            // System clock
    input wire reset_n,        // Active-low reset
    input wire motion_sensor,  // Motion detection (1 = motion detected)
    input wire obstruction,    // Obstruction sensor (1 = obstruction present)
    output reg door_open,      // Door control (1 = open, 0 = closed)
    output reg door_close      // Door control (1 = close, 0 = stop)
);

    // Parameters
    parameter TIMEOUT = 32'd50000000; // 1 second timeout @ 50MHz clock
    
    // Internal signals
    reg [31:0] timeout_counter;
    reg motion_detected;
    
    // Edge detection for motion sensor
    reg motion_prev;
    always @(posedge clk or negedge reset_n) begin
        if (!reset_n) begin
            motion_prev <= 1'b0;
            motion_detected <= 1'b0;
        end else begin
            motion_prev <= motion_sensor;
            motion_detected <= motion_sensor & !motion_prev;
        end
    end
    
    // Main control FSM
    reg [1:0] state;
    localparam IDLE      = 2'b00;
    localparam OPENING   = 2'b01;
    localparam OPEN      = 2'b10;
    localparam CLOSING   = 2'b11;
    
    always @(posedge clk or negedge reset_n) begin
        if (!reset_n) begin
            state <= IDLE;
            door_open <= 1'b0;
            door_close <= 1'b0;
            timeout_counter <= 32'b0;
        end else begin
            case (state)
                IDLE: begin
                    door_open <= 1'b0;
                    door_close <= 1'b0;
                    if (motion_detected) begin
                        state <= OPENING;
                        door_open <= 1'b1;
                    end
                end
                
                OPENING: begin
                    // Wait for door to fully open (simplified)
                    // In real implementation, would wait for limit switch
                    state <= OPEN;
                    timeout_counter <= TIMEOUT;
                end
                
                OPEN: begin
                    door_open <= 1'b0; // Stop opening command
                    
                    if (motion_detected) begin
                        // Reset timeout if motion detected again
                        timeout_counter <= TIMEOUT;
                    end else if (timeout_counter > 0) begin
                        timeout_counter <= timeout_counter - 1;
                    end else begin
                        if (!obstruction) begin
                            state <= CLOSING;
                            door_close <= 1'b1;
                        end
                    end
                end
                
                CLOSING: begin
                    if (obstruction || motion_detected) begin
                        // Stop closing if obstruction or new motion detected
                        state <= OPEN;
                        door_close <= 1'b0;
                        timeout_counter <= TIMEOUT;
                    end
                    // Wait for door to fully close (simplified)
                    // In real implementation, would wait for limit switch
                    else begin
                        state <= IDLE;
                        door_close <= 1'b0;
                    end
                end
            endcase
        end
    end

endmodule

# Testbench

module automatic_door_tb;
    reg clk;
    reg reset_n;
    reg motion_sensor;
    reg obstruction;
    wire door_open;
    wire door_close;
    
    automatic_door_controller dut (
        .clk(clk),
        .reset_n(reset_n),
        .motion_sensor(motion_sensor),
        .obstruction(obstruction),
        .door_open(door_open),
        .door_close(door_close)
    );
    
    // Clock generation
    initial begin
        clk = 0;
        forever #10 clk = ~clk;
    end
    
    // Test sequence
    initial begin
        // Initialize
        reset_n = 0;
        motion_sensor = 0;
        obstruction = 0;
        
        // Reset
        #20 reset_n = 1;
        
        // Test 1: Normal operation
        #30 motion_sensor = 1;  // Detect motion
        #20 motion_sensor = 0;
        #1000;                // Wait for timeout
        
        // Test 2: Motion during closing
        motion_sensor = 1;     // Detect motion again
        #20 motion_sensor = 0;
        #500;                  // Wait partial timeout
        motion_sensor = 1;     // Detect motion again
        #20 motion_sensor = 0;
        #1000;                // Wait for timeout
        
        // Test 3: Obstruction during closing
        motion_sensor = 1;     // Detect motion
        #20 motion_sensor = 0;
        #1000;                // Wait for timeout
        obstruction = 1;       // Create obstruction
        #100;
        obstruction = 0;
        
        #1000 $finish;
    end
    
    initial begin
        $dumpfile("door_controller.vcd");
        $dumpvars(0, automatic_door_tb);
    end
endmodule

# Implementation Notes
Clock Frequency: The design assumes a 50MHz clock (20ns period). The timeout value of 50,000,000 cycles corresponds to 1 second.

Motion Detection: The design includes edge detection to identify new motion events.

Safety Features:

The door won't close if an obstruction is detected

Any new motion detection while closing will reopen the door

Real-world Considerations:

In a real implementation, you would need limit switches to detect fully open/closed positions

The motor driver would likely use PWM for smooth operation

Additional debouncing might be needed for sensors

Possible Enhancements:

Add a "partially open" state for energy savings

Implement variable timeout based on time of day

Add emergency open/close controls

This RTL design provides a solid foundation for an automatic door controller that can be synthesized for an FPGA or ASIC implementation.
