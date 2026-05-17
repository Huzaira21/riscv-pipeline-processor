//================================ COMPLETE RISC-V
PIPELINED PROCESSOR ================================//
//..............................................Program Counter..............................//
module program_counter(
 input clk,
 input rst,
 input [31:0] pc_in,
 output reg [31:0] pc_out
);
 always @(posedge clk or posedge rst) begin
 if (rst) begin
 pc_out <= 32'b00;
 end else begin
 pc_out <= pc_in;
 end
 end
endmodule
//............................................PC Adder .......................................//
module pc_adder(
 input [31:0] pc_in,
 output [31:0] pc_next
);
 assign pc_next = pc_in + 4;
endmodule
//...........................................PC Mux(2x1).......................................//
module pc_mux(
 input [31:0] pc_in,
 input [31:0] pc_branch,
 input pc_select,
 output [31:0] pc_out
);
 assign pc_out = pc_select ? pc_branch : pc_in;
endmodule

//.........................................Instruction Memory................................//
module Instruction_Memory(rst, clk, read_address, instruction_out);
 input rst, clk;
 input [31:0] read_address;
 output [31:0] instruction_out;
 reg [31:0] I_Mem [63:0];
 integer k;

 assign instruction_out = I_Mem[read_address >> 2]; // Word addressing
 always @(posedge clk or posedge rst) begin
 if (rst) begin
 for (k = 0; k < 64; k = k + 1) begin
 I_Mem[k] = 32'b00;
 end
 end else begin
 // R-type instructions
 I_Mem[0] = 32'b00000000000000000000000000000000; // nop
 I_Mem[1] = 32'b0000000_11001_10000_000_01101_0110011; // add
x13, x16, x25
 I_Mem[2] = 32'b0100000_00011_01000_000_00101_0110011; // sub
x5, x8, x3
 I_Mem[3] = 32'b0000000_00011_00010_111_00001_0110011; // and
x1, x2, x3
 I_Mem[4] = 32'b0000000_00101_00011_110_00100_0110011; // or
x4, x3, x5
 I_Mem[5] = 32'b0000000_00101_00011_100_00100_0110011; // xor
x4, x3, x5
 I_Mem[6] = 32'b0000000_00101_00011_001_00100_0110011; // sll
x4, x3, x5
 I_Mem[7] = 32'b0000000_00101_00011_101_00100_0110011; // srl
x4, x3, x5
 I_Mem[8] = 32'b0100000_00010_00011_101_00101_0110011; // sra
x5, x3, x2
 I_Mem[9] = 32'b0000000_00010_00011_010_00101_0110011; // slt
x5, x3, x2

 // I-type instructions
 I_Mem[10] = 32'b000000000010_10101_000_10110_0010011; // addi
x22, x21, 2
I_Mem[11] = 32'b000000000011_01000_110_01001_0010011; // ori
x9, x8, 3
 I_Mem[12] = 32'b000000000100_01000_100_01001_0010011; // xori
x9, x8, 4
 I_Mem[13] = 32'b000000000101_00010_111_00001_0010011; // andi
x1, x2, 5
 I_Mem[14] = 32'b000000000110_00011_001_00100_0010011; // slli
x4, x3, 6
 I_Mem[15] = 32'b000000000111_00011_101_00100_0010011; // srli
x4, x3, 7
 I_Mem[16] = 32'b010000000111_00011_101_00101_0010011; // srai
x5, x3, 7
 I_Mem[17] = 32'b000000001001_00011_010_00101_0010011; // slti
x5, x3, 9

 // Load instructions
 I_Mem[18] = 32'b000000000101_00011_000_01001_0000011; // lb
x9, 5(x3)
 I_Mem[19] = 32'b000000000011_00011_001_01001_0000011; // lh
x9, 3(x3)
 I_Mem[20] = 32'b000000001111_00010_010_01000_0000011; // lw
x8, 15(x2)

 // Store instructions
 I_Mem[21] = 32'b0000000_01111_00011_000_01000_0100011; // sb
x15, 8(x3)
 I_Mem[22] = 32'b0000000_01110_00110_001_01010_0100011; // sh
x14, 10(x6)
 I_Mem[23] = 32'b0000000_01110_00110_010_01100_0100011; // sw
x14, 12(x6)

 // Branch instructions
 I_Mem[24] = 32'b0_000000_01001_01001_000_0110_0_1100011; //
beq x9, x9, 12
 I_Mem[25] = 32'b0_000000_01001_01001_001_0111_0_1100011; //
bne x9, x9, 14

 // U-type instructions
 I_Mem[26] = 32'b00000000000000101000_00011_0110111; // lui
x3, 40
 I_Mem[27] = 32'b00000000000000101000_00101_0010111; // auipc
x5, 40

// J-type instructions
 I_Mem[28] = 32'b0_0000000100_0_00000000_00001_1101111; // jal
x1, 8
 end
 end
endmodule
//...................................IF/ID register.......................................................
module IFID_Register(clk, rst, stall, PC_if, Instr_if, PC_id, Instr_id);
 input clk, rst, stall;
 input [31:0] PC_if, Instr_if;
 output reg [31:0] PC_id, Instr_id;
 always @(posedge clk or posedge rst) begin
 if (rst) begin
 PC_id <= 32'b00;
 Instr_id <= 32'b00;
 end
 else if (!stall) begin // Only update if not stalled
 PC_id <= PC_if;
 Instr_id <= Instr_if;
 end
 end
endmodule
//................................................Register File............................................//
module Register_File(clk, rst, RegWrite, Rs1, Rs2, Rd, Write_data,
read_data1, read_data2);
 input clk, rst, RegWrite;
 input [4:0] Rs1, Rs2, Rd;
 input [31:0] Write_data;
 output [31:0] read_data1, read_data2;
 reg [31:0] Registers [31:0];
 // Initialize registers
 initial begin
 Registers[0] = 0; Registers[1] = 3; Registers[2] = 2; Registers[3] =
12;
 Registers[4] = 20; Registers[5] = 3; Registers[6] = 44; Registers[7] =4;
Registers[8] = 2; Registers[9] = 1; Registers[10] = 23; Registers[11] =
4;
 Registers[12] = 90; Registers[13] = 10; Registers[14] = 20;
Registers[15] = 30;
 Registers[16] = 40; Registers[17] = 50; Registers[18] = 60;
Registers[19] = 70;
 Registers[20] = 80; Registers[21] = 80; Registers[22] = 90;
Registers[23] = 70;
 Registers[24] = 60; Registers[25] = 65; Registers[26] = 4; Registers[27]
= 32;
 Registers[28] = 12; Registers[29] = 34; Registers[30] = 5; Registers[31]
= 10;
 end
 integer k;
 always @(posedge clk) begin
 if (rst) begin
 for (k = 1; k < 32; k = k + 1) begin // x0 always 0
 Registers[k] <= 32'b00;
 end
 end
 else if (RegWrite && (Rd != 5'b00000)) begin // Don't write to x0
 Registers[Rd] <= Write_data;
 end
 end
 assign read_data1 = Registers[Rs1];
 assign read_data2 = Registers[Rs2];
endmodule
//............................................Main Control Unit...............................................//
module main_control_unit(
 input [6:0] opcode,
 output reg RegWrite,
 output reg MemRead,
 output reg MemWrite,
 output reg MemToReg,
 output reg ALUSrc,
 output reg Branch,
 output reg Jump,
 output reg [1:0] ALUOp
)
always @(*) begin
 case (opcode)
 7'b0110011: begin // R-type
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b0, 1'b0, 1'b1, 1'b0, 1'b0, 1'b0, 1'b0, 2'b10};
 end
 7'b0010011: begin // I-type
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b1, 1'b0, 1'b1, 1'b0, 1'b0, 1'b0, 1'b0, 2'b10};
 end
 7'b0000011: begin // Load
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b1, 1'b1, 1'b1, 1'b1, 1'b0, 1'b0, 1'b0, 2'b00};
 end
 7'b0100011: begin // Store
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b1, 1'b0, 1'b0, 1'b0, 1'b1, 1'b0, 1'b0, 2'b00};
 end
 7'b1100011: begin // Branch
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b0, 1'b0, 1'b0, 1'b0, 1'b0, 1'b1, 1'b0, 2'b01};
 end
 7'b1101111: begin // Jump (JAL)
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b0, 1'b0, 1'b1, 1'b0, 1'b0, 1'b0, 1'b1, 2'b00};
 end
 7'b0110111: begin // LUI
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b1, 1'b0, 1'b1, 1'b0, 1'b0, 1'b0, 1'b0, 2'b11};
 end
 7'b0010111: begin // AUIPC
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b1, 1'b0, 1'b1, 1'b0, 1'b0, 1'b0, 1'b0, 2'b00};
 end
 default: begin
 {ALUSrc, MemToReg, RegWrite, MemRead, MemWrite, Branch,
Jump, ALUOp} = {1'b0, 1'b0, 1'b0, 1'b0, 1'b0, 1'b0, 1'b0, 2'b00};
 end
 endcase
 end
endmodule


//......................................................Immediate Generator..........................//
module immediate_generator(
 input [31:0] instruction,
 output reg [31:0] imm_out
);
 always @(*) begin
 case (instruction[6:0])
 7'b0010011: imm_out = {{20{instruction[31]}}, instruction[31:20]}; //
I-type
 7'b0000011: imm_out = {{20{instruction[31]}}, instruction[31:20]}; //
Load-type
 7'b0100011: imm_out = {{20{instruction[31]}}, instruction[31:25],
instruction[11:7]}; // Store-type
 7'b1100011: imm_out = {{19{instruction[31]}}, instruction[7],
instruction[30:25], instruction[11:8], 1'b0}; // B-type
 7'b0110111: imm_out = {instruction[31:12], 12'b0}; // U-type (LUI)
 7'b0010111: imm_out = {instruction[31:12], 12'b0}; // U-type
(AUIPC)
 7'b1101111: imm_out = {{11{instruction[31]}}, instruction[19:12],
instruction[20], instruction[30:21], 1'b0}; // J-type
 default: imm_out = 32'b0; // Default case
 endcase
 end
endmodule
//..............................................Hazard Detection Unit..................................//
module Hazard_Detection_Unit(
 input [4:0] Rs1_ID, // Source register 1 in ID stage
 input [4:0] Rs2_ID, // Source register 2 in ID stage
 input [4:0] Rd_EX, // Destination register in EX stage
 input MemRead_EX, // MemRead signal in EX stage
 input Branch_ID, // Branch signal in ID stage
 input Jump_ID, // Jump signal in ID stage
 output reg PCWrite, // PC write enable
 output reg IFID_Write, // IF/ID register write enable
 output reg Control_Mux // Control mux select (0: normal, 1: bubble)
);
 always @(*) begin
 // Default: no stall
 PCWrite = 1'b1;
 IFID_Write = 1'b1;
 Control_Mux = 1'b0;
// Load-use hazard detection
 if (MemRead_EX &&
 ((Rd_EX == Rs1_ID && Rs1_ID != 5'b00000) ||
 (Rd_EX == Rs2_ID && Rs2_ID != 5'b00000))) begin
 PCWrite = 1'b0; // Stall PC
 IFID_Write = 1'b0; // Stall IF/ID register
 Control_Mux = 1'b1; // Insert bubble (nop) in EX stage
 end

 // Control hazard detection (branch/jump)
 if (Branch_ID || Jump_ID) begin
 // Could implement branch prediction here
 // For now, we assume control hazards are handled by flushing
 end
 end
endmodule
//................................................................................................ID/EX
Register.................................................................................................................
....................
module IDEXE_Register(
 clk, rst, flush,
 RegWrite_id, MemRead_id, MemWrite_id, MemToReg_id, ALUSrc_id,
Branch_id, Jump_id, PC_id, ALUOp_id,
 read_data1_id, read_data2_id, imm_out_id, Rs1_id, Rs2_id, Rd_id, Instr_id,
 RegWrite_ex, MemRead_ex, MemWrite_ex, MemToReg_ex, ALUSrc_ex,
Branch_ex, Jump_ex, PC_ex, ALUOp_ex,
 read_data1_ex, read_data2_ex, imm_out_ex, Rs1_ex, Rs2_ex, Rd_ex,
Instr_ex
);
 input clk, rst, flush;
 input RegWrite_id, MemRead_id, MemWrite_id, MemToReg_id,
ALUSrc_id, Branch_id, Jump_id;
 input [31:0] PC_id, read_data1_id, read_data2_id, imm_out_id, Instr_id;
 input [1:0] ALUOp_id;
 input [4:0] Rs1_id, Rs2_id, Rd_id;
 output reg RegWrite_ex, MemRead_ex, MemWrite_ex, MemToReg_ex,
ALUSrc_ex, Branch_ex, Jump_ex;
 output reg [31:0] PC_ex, read_data1_ex, read_data2_ex, imm_out_ex,
Instr_ex;


output reg [1:0] ALUOp_ex;
 output reg [4:0] Rs1_ex, Rs2_ex, Rd_ex;
 always @(posedge clk or posedge rst) begin
 if (rst || flush) begin
 RegWrite_ex <= 1'b0;
 MemRead_ex <= 1'b0;
 MemWrite_ex <= 1'b0;
 MemToReg_ex <= 1'b0;
 ALUSrc_ex <= 1'b0;
 Branch_ex <= 1'b0;
 Jump_ex <= 1'b0;
 PC_ex <= 32'b00;
 read_data1_ex <= 32'b00;
 read_data2_ex <= 32'b00;
 imm_out_ex <= 32'b00;
 Rs1_ex <= 5'b00;
 Rs2_ex <= 5'b00;
 Rd_ex <= 5'b00;
 Instr_ex <= 32'b00;
 ALUOp_ex <= 2'b00;
 end else begin
 RegWrite_ex <= RegWrite_id;
 MemRead_ex <= MemRead_id;
 MemWrite_ex <= MemWrite_id;
 MemToReg_ex <= MemToReg_id;
 ALUSrc_ex <= ALUSrc_id;
 Branch_ex <= Branch_id;
 Jump_ex <= Jump_id;
 PC_ex <= PC_id;
 read_data1_ex <= read_data1_id;
 read_data2_ex <= read_data2_id;
 imm_out_ex <= imm_out_id;
 Rs1_ex <= Rs1_id;
 Rs2_ex <= Rs2_id;
 Rd_ex <= Rd_id;
 Instr_ex <= Instr_id;
 ALUOp_ex <= ALUOp_id;
 end
 end
endmodule
module ALU(
 input [31:0] A,
 input [31:0] B,
 input [3:0] ALUcontrol_In,
 output reg [31:0] Result,
 output reg Zero
);
 always @(*) begin
 case (ALUcontrol_In)
 4'b0000: Result = A + B; // ADD
 4'b0001: Result = A - B; // SUB
 4'b0010: Result = A & B; // AND
 4'b0011: Result = A | B; // OR
 4'b0100: Result = A ^ B; // XOR
 4'b0101: Result = A << B[4:0]; // SLL
 4'b0110: Result = A >> B[4:0]; // SRL
 4'b0111: Result = $signed(A) >>> B[4:0]; // SRA
 4'b1000: Result = ($signed(A) < $signed(B)) ? 32'b1 : 32'b0; // SLT
 4'b1001: Result = B; // Pass B (for LUI)
 default: Result = 32'b0;
 endcase
 Zero = (Result == 32'b0) ? 1 : 0;
 end
endmodule
//..............................................ALU Control...................................................//
module ALU_Control(
 input [2:0] funct3,
 input [6:0] funct7,
 input [1:0] ALUOp,
 output reg [3:0] ALUcontrol_Out
);
 always @(*) begin
 case (ALUOp)
 2'b00: ALUcontrol_Out = 4'b0000; // ADD (for load/store/AUIPC)
 2'b01: ALUcontrol_Out = 4'b0001; // SUB (for branch)
 2'b11: ALUcontrol_Out = 4'b1001; // Pass B (for LUI)
 2'b10: begin // R-type and I-type
 case ({funct7, funct3})
 10'b0000000_000: ALUcontrol_Out = 4'b0000; // ADD
 10'b0100000_000: ALUcontrol_Out = 4'b0001; // SUB
 10'b0000000_111: ALUcontrol_Out = 4'b0010; // AND
 10'b0000000_110: ALUcontrol_Out = 4'b0011; // OR
 10'b0000000_100: ALUcontrol_Out = 4'b0100; // XOR
 10'b0000000_001: ALUcontrol_Out = 4'b0101; // SLL
 10'b0000000_101: ALUcontrol_Out = 4'b0110; // SRL
 10'b0100000_101: ALUcontrol_Out = 4'b0111; // SRA
 10'b0000000_010: ALUcontrol_Out = 4'b1000; // SLT
default: ALUcontrol_Out = 4'b0000;
 endcase
 end
 default: ALUcontrol_Out = 4'b0000;
 endcase
 end
endmodule
//................................................ALU Mux(2x1)......................................//
module MUX2to1(
 input [31:0] input0,
 input [31:0] input1,
 input select,
 output [31:0] out
);
 assign out = (select) ? input1 : input0;
endmodule
//..............................................Forwarding Unit..................................//
module Forwarding_Unit(
 input [4:0] Rs1_EX, // Source register 1 in EX stage
 input [4:0] Rs2_EX, // Source register 2 in EX stage
 input [4:0] Rd_MEM, // Destination register in MEM stage
 input [4:0] Rd_WB, // Destination register in WB stage
 input RegWrite_MEM, // RegWrite signal in MEM stage
 input RegWrite_WB, // RegWrite signal in WB stage
 output reg [1:0] ForwardA, // Forwarding control for ALU input A
 output reg [1:0] ForwardB // Forwarding control for ALU input B
);
 always @(*) begin
 // Default: no forwarding
 ForwardA = 2'b00;
 ForwardB = 2'b00;
// EX/MEM forwarding (highest priority)
 if (RegWrite_MEM && (Rd_MEM != 5'b00000)) begin
 if (Rd_MEM == Rs1_EX) begin
 ForwardA = 2'b10; // Forward from MEM stage
 end
 if (Rd_MEM == Rs2_EX) begin
 ForwardB = 2'b10; // Forward from MEM stage
 end
 end

 // MEM/WB forwarding (lower priority)
 if (RegWrite_WB && (Rd_WB != 5'b00000)) begin
 if ((Rd_WB == Rs1_EX) && (ForwardA != 2'b10)) begin
 ForwardA = 2'b01; // Forward from WB stage
 end
 if ((Rd_WB == Rs2_EX) && (ForwardB != 2'b10)) begin
 ForwardB = 2'b01; // Forward from WB stage
 end
 end
 end
endmodule
// 3-to-1 Mux for Forwarding
module MUX3to1_Forward(
 input [31:0] input0, // No forwarding
 input [31:0] input1, // Forward from WB
 input [31:0] input2, // Forward from MEM
 input [1:0] select, // 00: no forward, 01: WB, 10: MEM
 output reg [31:0] out
);
 always @(*) begin
 case(select)
 2'b00: out = input0;
 2'b01: out = input1;
 2'b10: out = input2;
 default: out = input0;
 endcase
 end
endmodule
//...............................................EX/MEM
Register...................................................//
module EXMEM_Register(
 clk, rst,
 RegWrite_ex, MemRead_ex, MemWrite_ex, MemToReg_ex, Branch_ex,
Jump_ex,
 ALU_Result, ALU_Zero, read_data2_ex, Rd_ex, branch_target,
PC_plus4_ex,
 RegWrite_mem, MemRead_mem, MemWrite_mem, MemToReg_mem,
Branch_mem, Jump_mem,
 ALU_Result_mem, ALU_Zero_mem, read_data2_mem, Rd_mem,
branch_target_mem, PC_plus4_mem
);
 input clk, rst, RegWrite_ex, MemRead_ex, MemWrite_ex, MemToReg_ex,
Branch_ex, Jump_ex, ALU_Zero;
 input [31:0] ALU_Result, branch_target, read_data2_ex, PC_plus4_ex;
 input [4:0] Rd_ex;
 output reg RegWrite_mem, MemRead_mem, MemWrite_mem,
MemToReg_mem, Branch_mem, Jump_mem, ALU_Zero_mem;
 output reg [31:0] ALU_Result_mem, branch_target_mem,
read_data2_mem, PC_plus4_mem;
 output reg [4:0] Rd_mem;
 always @(posedge clk or posedge rst) begin
 if (rst) begin
 RegWrite_mem <= 1'b0;
 MemRead_mem <= 1'b0;
 MemWrite_mem <= 1'b0;
 MemToReg_mem <= 1'b0;
 Branch_mem <= 1'b0;
 Jump_mem <= 1'b0;
 ALU_Zero_mem <= 1'b0;
 ALU_Result_mem <= 32'b00;
 branch_target_mem <= 32'b00;
 read_data2_mem <= 32'b00;
 PC_plus4_mem <= 32'b00;
 Rd_mem <= 5'b00;
 end
 else begin
 RegWrite_mem <= RegWrite_ex;
 MemRead_mem <= MemRead_ex;
 MemWrite_mem <= MemWrite_ex;
 MemToReg_mem <= MemToReg_ex;
Branch_mem <= Branch_ex;
 Jump_mem <= Jump_ex;
 ALU_Zero_mem <= ALU_Zero;
 ALU_Result_mem <= ALU_Result;
 branch_target_mem <= branch_target;
 read_data2_mem <= read_data2_ex;
 PC_plus4_mem <= PC_plus4_ex;
 Rd_mem <= Rd_ex;
 end
 end
endmodule
//================================ MISSING PARTS OF RISC-V
PROCESSOR ================================//
//................................................Data Memory
(Completed)......................................//
module Data_Memory(
 input clk,
 input rst,
 input MemRead,
 input MemWrite,
 input [31:0] address,
 input [31:0] write_data,
 output [31:0] read_data
);
 reg [31:0] D_Memory [63:0];
 integer k;
 assign read_data = (MemRead) ? D_Memory[address >> 2] : 32'b00; //
Word addressing
 always @(posedge clk or posedge rst) begin
 if (rst) begin
 for (k = 0; k < 64; k = k + 1) begin
 D_Memory[k] <= 32'b00;
 end
 end else if (MemWrite) begin
 D_Memory[address >> 2] <= write_data;
 end
 end
endmodule
//...............................................MEM/WB
Register...................................................//
module MEMWB_Register(
 clk, rst,
 RegWrite_mem, MemToReg_mem,
 ALU_Result_mem, read_data_mem, Rd_mem, PC_plus4_mem,
 RegWrite_wb, MemToReg_wb,
 ALU_Result_wb, read_data_wb, Rd_wb, PC_plus4_wb
);
 input clk, rst, RegWrite_mem, MemToReg_mem;
 input [31:0] ALU_Result_mem, read_data_mem, PC_plus4_mem;
 input [4:0] Rd_mem;
 output reg RegWrite_wb, MemToReg_wb;
 output reg [31:0] ALU_Result_wb, read_data_wb, PC_plus4_wb;
 output reg [4:0] Rd_wb;
 always @(posedge clk or posedge rst) begin
 if (rst) begin
 RegWrite_wb <= 1'b0;
 MemToReg_wb <= 1'b0;
 ALU_Result_wb <= 32'b00;
 read_data_wb <= 32'b00;
 PC_plus4_wb <= 32'b00;
 Rd_wb <= 5'b00;
 end
 else begin
 RegWrite_wb <= RegWrite_mem;
 MemToReg_wb <= MemToReg_mem;
 ALU_Result_wb <= ALU_Result_mem;
 read_data_wb <= read_data_mem;
 PC_plus4_wb <= PC_plus4_mem;
 Rd_wb <= Rd_mem;
 end
 end
endmodule
//..............................................Branch Adder..................................//
module branch_adder(
 input [31:0] pc_in,
 input [31:0] imm_in,
output [31:0] branch_target
);
 assign branch_target = pc_in + imm_in;
endmodule
//..............................................Branch Control Unit..................................//
module branch_control(
 input [2:0] funct3,
 input Zero,
 input Branch,
 input Jump,
 output reg PCSrc
);
 always @(*) begin
 if (Jump) begin
 PCSrc = 1'b1; // Jump always taken
 end else if (Branch) begin
 case (funct3)
 3'b000: PCSrc = Zero; // BEQ
 3'b001: PCSrc = ~Zero; // BNE
 default: PCSrc = 1'b0;
 endcase
 end else begin
 PCSrc = 1'b0; // No branch/jump
 end
 end
endmodule
//..............................................Write Back Mux..................................//
module writeback_mux(
 input [31:0] ALU_Result,
 input [31:0] read_data,
 input [31:0] PC_plus4,
 input MemToReg,
 input Jump,
 output [31:0] write_data
);
 assign write_data = Jump ? PC_plus4 : (MemToReg ? read_data :
ALU_Result);
endmodule
//..............................................Control Mux (for hazard
handling)..................................//
module control_mux(
 input RegWrite_in, MemRead_in, MemWrite_in, MemToReg_in,
ALUSrc_in, Branch_in, Jump_in,
 input [1:0] ALUOp_in,
 input Control_Mux_Select,
 output RegWrite_out, MemRead_out, MemWrite_out, MemToReg_out,
ALUSrc_out, Branch_out, Jump_out,
 output [1:0] ALUOp_out
);
 assign RegWrite_out = Control_Mux_Select ? 1'b0 : RegWrite_in;
 assign MemRead_out = Control_Mux_Select ? 1'b0 : MemRead_in;
 assign MemWrite_out = Control_Mux_Select ? 1'b0 : MemWrite_in;
 assign MemToReg_out = Control_Mux_Select ? 1'b0 : MemToReg_in;
 assign ALUSrc_out = Control_Mux_Select ? 1'b0 : ALUSrc_in;
 assign Branch_out = Control_Mux_Select ? 1'b0 : Branch_in;
 assign Jump_out = Control_Mux_Select ? 1'b0 : Jump_in;
 assign ALUOp_out = Control_Mux_Select ? 2'b00 : ALUOp_in;
endmodule
//..............................................Top Level Processor..................................//
module RISC_V_Processor(
 input clk,
 input rst
);
 // Wire declarations
 wire [31:0] PC, PC_next, PC_plus4, PC_branch, instruction;
 wire [31:0] PC_id, instruction_id;
 wire [31:0] read_data1, read_data2, imm_out;
 wire [31:0] ALU_input_A, ALU_input_B, ALU_Result;
 wire [31:0] read_data1_ex, read_data2_ex, imm_out_ex, PC_ex,
instruction_ex;
 wire [31:0] ALU_Result_mem, read_data2_mem, branch_target_mem,
PC_plus4_mem;
 wire [31:0] ALU_Result_wb, read_data_wb, PC_plus4_wb;
 wire [31:0] write_data, data_memory_out, branch_target;
 wire [4:0] Rs1, Rs2, Rd, Rs1_ex, Rs2_ex, Rd_ex, Rd_mem, Rd_wb;
 wire [3:0] ALU_control;
 wire [2:0] funct3;
 wire [6:0] funct7, opcode;
 wire [1:0] ALUOp, ALUOp_ex, ForwardA, ForwardB;
 wire RegWrite, MemRead, MemWrite, MemToReg, ALUSrc, Branch,
Jump, Zero;
 wire RegWrite_ex, MemRead_ex, MemWrite_ex, MemToReg_ex,
ALUSrc_ex, Branch_ex, Jump_ex;
 wire RegWrite_mem, MemRead_mem, MemWrite_mem,
MemToReg_mem, Branch_mem, Jump_mem, Zero_mem;
 wire RegWrite_wb, MemToReg_wb;
 wire PCSrc, PCWrite, IFID_Write, Control_Mux_Select;
 wire RegWrite_ctrl, MemRead_ctrl, MemWrite_ctrl, MemToReg_ctrl,
ALUSrc_ctrl, Branch_ctrl, Jump_ctrl;
 wire [1:0] ALUOp_ctrl;
 // Instruction field extraction
 assign opcode = instruction_id[6:0];
 assign Rs1 = instruction_id[19:15];
 assign Rs2 = instruction_id[24:20];
 assign Rd = instruction_id[11:7];
 assign funct3 = instruction_ex[14:12];
 assign funct7 = instruction_ex[31:25];
 // IF Stage
 program_counter PC_reg(
 .clk(clk),
 .rst(rst),
 .pc_in(PC_branch),
 .pc_out(PC)
 );
 pc_adder PC_Add(
 .pc_in(PC),
 .pc_next(PC_plus4)
 );
 pc_mux PC_Mux(
 .pc_in(PC_plus4),
 .pc_branch(branch_target_mem),
 .pc_select(PCSrc),
 .pc_out(PC_next)
 );
 assign PC_branch = PCWrite ? PC_next : PC;
 Instruction_Memory IM(
 .rst(rst),
 .clk(clk),
 .read_address(PC),
 .instruction_out(instruction)
 );
 // IF/ID Register
 IFID_Register IFID(
 .clk(clk),
 .rst(rst),
 .stall(~IFID_Write),
 .PC_if(PC),
 .Instr_if(instruction),
 .PC_id(PC_id),
 .Instr_id(instruction_id)
 );
 // ID Stage
 Register_File RF(
 .clk(clk),
 .rst(rst),
 .RegWrite(RegWrite_wb),
 .Rs1(Rs1),
 .Rs2(Rs2),
 .Rd(Rd_wb),
 .Write_data(write_data),
 .read_data1(read_data1),
 .read_data2(read_data2)
 );
 main_control_unit MCU(
 .opcode(opcode),
 .RegWrite(RegWrite_ctrl),
 .MemRead(MemRead_ctrl),
 .MemWrite(MemWrite_ctrl),
 .MemToReg(MemToReg_ctrl),
 .ALUSrc(ALUSrc_ctrl),
 .Branch(Branch_ctrl),
 .Jump(Jump_ctrl),
 .ALUOp(ALUOp_ctrl)
 );
control_mux CTRL_MUX(
 .RegWrite_in(RegWrite_ctrl),
 .MemRead_in(MemRead_ctrl),
 .MemWrite_in(MemWrite_ctrl),
 .MemToReg_in(MemToReg_ctrl),
 .ALUSrc_in(ALUSrc_ctrl),
 .Branch_in(Branch_ctrl),
 .Jump_in(Jump_ctrl),
 .ALUOp_in(ALUOp_ctrl),
 .Control_Mux_Select(Control_Mux_Select),
 .RegWrite_out(RegWrite),
 .MemRead_out(MemRead),
 .MemWrite_out(MemWrite),
 .MemToReg_out(MemToReg),
 .ALUSrc_out(ALUSrc),
 .Branch_out(Branch),
 .Jump_out(Jump),
 .ALUOp_out(ALUOp)
 );
 immediate_generator IG(
 .instruction(instruction_id),
 .imm_out(imm_out)
 );
 Hazard_Detection_Unit HDU(
 .Rs1_ID(Rs1),
 .Rs2_ID(Rs2),
 .Rd_EX(Rd_ex),
 .MemRead_EX(MemRead_ex),
 .Branch_ID(Branch),
 .Jump_ID(Jump),
 .PCWrite(PCWrite),
 .IFID_Write(IFID_Write),
 .Control_Mux(Control_Mux_Select)
 );
 // ID/EX Register
 IDEXE_Register IDEX(
 .clk(clk),
 .rst(rst),
.flush(PCSrc),
 .RegWrite_id(RegWrite),

.MemRead_id(MemRead),
 .MemWrite_id(MemWrite),
 .MemToReg_id(MemToReg),
 .ALUSrc_id(ALUSrc),
 .Branch_id(Branch),
 .Jump_id(Jump),
 .PC_id(PC_id),
 .ALUOp_id(ALUOp),
 .read_data1_id(read_data1),
 .read_data2_id(read_data2),
 .imm_out_id(imm_out),
 .Rs1_id(Rs1),
 .Rs2_id(Rs2),
 .Rd_id(Rd),
 .Instr_id(instruction_id),
 .RegWrite_ex(RegWrite_ex),
 .MemRead_ex(MemRead_ex),
 .MemWrite_ex(MemWrite_ex),
 .MemToReg_ex(MemToReg_ex),
 .ALUSrc_ex(ALUSrc_ex),
 .Branch_ex(Branch_ex),
 .Jump_ex(Jump_ex),
 .PC_ex(PC_ex),
 .ALUOp_ex(ALUOp_ex),
 .read_data1_ex(read_data1_ex),
 .read_data2_ex(read_data2_ex),
 .imm_out_ex(imm_out_ex),
 .Rs1_ex(Rs1_ex),
 .Rs2_ex(Rs2_ex),
 .Rd_ex(Rd_ex),
 .Instr_ex(instruction_ex)
 );
 // EX Stage
 Forwarding_Unit FU(
 .Rs1_EX(Rs1_ex),
 .Rs2_EX(Rs2_ex),
 .Rd_MEM(Rd_mem),
 .Rd_WB(Rd_wb),
 .RegWrite_MEM(RegWrite_mem),
RegWrite_WB(RegWrite_wb),
 .ForwardA(ForwardA),
 .ForwardB(ForwardB)
 );
 MUX3to1_Forward Forward_A_Mux(
 .input0(read_data1_ex),
 .input1(write_data),
 .input2(ALU_Result_mem),
 .select(ForwardA),
 .out(ALU_input_A)
 );
 MUX3to1_Forward Forward_B_Mux(
 .input0(read_data2_ex),
 .input1(write_data),
 .input2(ALU_Result_mem),
 .select(ForwardB),
 .out(ALU_input_B)
 );
 MUX2to1 ALU_Mux(
 .input0(ALU_input_B),
 .input1(imm_out_ex),
 .select(ALUSrc_ex),
 .out(ALU_input_B)
 );
 ALU_Control ALU_CTRL(
 .funct3(funct3),
 .funct7(funct7),
 .ALUOp(ALUOp_ex),
 .ALUcontrol_Out(ALU_control)
 );
 ALU ALU_Unit(
 .A(ALU_input_A),
 .B(ALU_input_B),
 .ALUcontrol_In(ALU_control),
 .Result(ALU_Result),
 .Zero(Zero)
 );
 branch_adder Branch_Add(
 .pc_in(PC_ex),
 .imm_in(imm_out_ex),
 .branch_target(branch_target)
 );
 // EX/MEM Register
 EXMEM_Register EXMEM(
 .clk(clk),
 .rst(rst),
 .RegWrite_ex(RegWrite_ex),
 .MemRead_ex(MemRead_ex),
 .MemWrite_ex(MemWrite_ex),
 .MemToReg_ex(MemToReg_ex),
 .Branch_ex(Branch_ex),
 .Jump_ex(Jump_ex),
 .ALU_Result(ALU_Result),
 .ALU_Zero(Zero),
 .read_data2_ex(read_data2_ex),
 .Rd_ex(Rd_ex),
 .branch_target(branch_target),
 .PC_plus4_ex(PC_ex + 4),
 .RegWrite_mem(RegWrite_mem),
 .MemRead_mem(MemRead_mem),
 .MemWrite_mem(MemWrite_mem),
 .MemToReg_mem(MemToReg_mem),
 .Branch_mem(Branch_mem),
 .Jump_mem(Jump_mem),
 .ALU_Result_mem(ALU_Result_mem),
 .ALU_Zero_mem(Zero_mem),
 .read_data2_mem(read_data2_mem),
 .Rd_mem(Rd_mem),
 .branch_target_mem(branch_target_mem),
 .PC_plus4_mem(PC_plus4_mem)
 );
 Data_Memory DM(
 .clk(clk),
 .rst(rst),
 .MemRead(MemRead_mem),
 .MemWrite(MemWrite_mem),
 .address(ALU_Result_mem)
.write_data(read_data2_mem),
 .read_data(data_memory_out)
 );
 branch_control BC(
 .funct3(instruction_ex[14:12]),
 .Zero(Zero_mem),
 .Branch(Branch_mem),
 .Jump(Jump_mem),
 .PCSrc(PCSrc)
 );
 MEMWB_Register MEMWB(
 .clk(clk),
 .rst(rst),
 .RegWrite_mem(RegWrite_mem),
 .MemToReg_mem(MemToReg_mem),
 .ALU_Result_mem(ALU_Result_mem),
 .read_data_mem(data_memory_out),
 .Rd_mem(Rd_mem),
 .PC_plus4_mem(PC_plus4_mem),
 .RegWrite_wb(RegWrite_wb),
 .MemToReg_wb(MemToReg_wb),
 .ALU_Result_wb(ALU_Result_wb),
 .read_data_wb(read_data_wb),
 .Rd_wb(Rd_wb),
 .PC_plus4_wb(PC_plus4_wb)
 );
 writeback_mux WB_Mux(
 .ALU_Result(ALU_Result_wb),
 .read_data(read_data_wb),
 .PC_plus4(PC_plus4_wb),
 .MemToReg(MemToReg_wb),
 .Jump(Jump_mem), // Use Jump from MEM stage for JAL
 .write_data(write_data)
 );
endmodule
module RISC_V_Processor_TB;
 reg clk;
reg rst;

 RISC_V_Processor uut (
 .clk(clk),
 .rst(rst)
 );

 initial begin
 clk = 0;
 forever #5 clk = ~clk;
 end

 initial begin
 rst = 1;
 #20;
 rst = 0;

 #500;

 $display("Test completed");
 $finish;
 end
endmodule