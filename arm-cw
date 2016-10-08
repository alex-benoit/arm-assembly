; 	Answers
;	1. My code can measure a maximum frequency of approximately 3.57MHz (period of 28 cycles) reliably. 
;	2. As simulation time increases the fractional error decreases.
;-------------------------------------------------------------------------------------------
;-------------------------------------------------------------------------------------------

; Standard definitions of Mode bits and Interrupt (I & F) flags in PSRs

Mode_USR        EQU     0x10
Mode_FIQ        EQU     0x11
Mode_IRQ        EQU     0x12
Mode_SVC        EQU     0x13
Mode_ABT        EQU     0x17
Mode_UND        EQU     0x1B
Mode_SYS        EQU     0x1F

I_Bit           EQU     0x80            	; when I bit is set, IRQ is disabled
F_Bit           EQU     0x40            	; when F bit is set, FIQ is disabled


;// <h> Stack Configuration (Stack Sizes in Bytes)
;//   <o0> Undefined Mode      <0x0-0xFFFFFFFF:8>
;//   <o1> Supervisor Mode     <0x0-0xFFFFFFFF:8>
;//   <o2> Abort Mode          <0x0-0xFFFFFFFF:8>
;//   <o3> Fast Interrupt Mode <0x0-0xFFFFFFFF:8>
;//   <o4> Interrupt Mode      <0x0-0xFFFFFFFF:8>
;//   <o5> User/System Mode    <0x0-0xFFFFFFFF:8>
;// </h>

UND_Stack_Size  EQU     0x00000000
SVC_Stack_Size  EQU     0x00000080
ABT_Stack_Size  EQU     0x00000000
FIQ_Stack_Size  EQU     0x00000000
IRQ_Stack_Size  EQU     0x00000080
USR_Stack_Size  EQU     0x00000000

ISR_Stack_Size  EQU     (UND_Stack_Size + SVC_Stack_Size + ABT_Stack_Size + FIQ_Stack_Size + IRQ_Stack_Size)

        	AREA    RESET, CODE
		ENTRY

Vectors         LDR     PC, Reset_Addr         
                LDR     PC, Undef_Addr
                LDR     PC, SWI_Addr
                LDR     PC, PAbt_Addr
                LDR     PC, DAbt_Addr
                NOP                           	; Reserved Vector 
                LDR     PC, IRQ_Addr
;               LDR     PC, [PC, #-0x0FF0]     	; Vector from VicVectAddr
                LDR     PC, FIQ_Addr

ACBASE		DCD	P0COUNT
SCONTR		DCD	SIMCONTROL

Reset_Addr      DCD     Reset_Handler
Undef_Addr      DCD     Undef_Handler
SWI_Addr        DCD     SWI_Handler
PAbt_Addr       DCD     PAbt_Handler
DAbt_Addr       DCD     DAbt_Handler
                DCD     0                      	; Reserved Address 
FIQ_Addr        DCD     FIQ_Handler

Undef_Handler   B       Undef_Handler
SWI_Handler     B       SWI_Handler
PAbt_Handler    B       PAbt_Handler
DAbt_Handler    B       DAbt_Handler
FIQ_Handler     B       FIQ_Handler

AREA 	ARMuser, CODE,READONLY

IRQ_Addr        DCD     ISR_FUNC1
EINT2		EQU 	16
Addr_VicIntEn	DCD	0xFFFFF010	 	; set to (1<<EINT0)
Addr_EXTMODE	DCD 	0xE01FC148   		; set to 1
Addr_PINSEL0	DCD	0xE002C000		; set to 2_1100
Addr_EXTINT	DCD	0xE01FC140

;  Addresses of two registers that allow faster input
Addr_IOPIN	DCD	0xE0028000
Addr_FIOMASK	DCD	0x3FFFC010
Addr_FIOPIN	DCD	0x3FFFC014

; Initialise the Interrupt System
ISR_FUNC1	STMED	R13!, {R0,R1}
			MOV 	R0, #(1 << 2) 	; bit 2 of EXTINT
			LDR 	R1, Addr_EXTINT	   
			STR	R0, [R1]	; EINT2 reset interrupt
			LDMED	R13!, {R0,R1}
			B 	ISR_FUNC

Reset_Handler
; PORT0.1 1->0 triggers EINT0 IRQ interrupt
				MOV R0, #(1 << EINT2)
				LDR R1, Addr_VicIntEn
				STR R0, [R1]
				MOV R0, #(1 << 30)
				LDR R1, Addr_PINSEL0
				STR R0, [R1]
				MOV R0, #(1 << 2)
				LDR R1, Addr_EXTMODE
				STR R0, [R1]

;  Setup Stack for each mode
                LDR     R0, =Stack_Top

;  Enter Undefined Instruction Mode and set its Stack Pointer
                MSR     CPSR_c, #Mode_UND:OR:I_Bit:OR:F_Bit
                MOV     SP, R0
                SUB     R0, R0, #UND_Stack_Size

;  Enter Abort Mode and set its Stack Pointer
                MSR     CPSR_c, #Mode_ABT:OR:I_Bit:OR:F_Bit
                MOV     SP, R0
                SUB     R0, R0, #ABT_Stack_Size

;  Enter FIQ Mode and set its Stack Pointer
                MSR     CPSR_c, #Mode_FIQ:OR:I_Bit:OR:F_Bit
                MOV     SP, R0
                SUB     R0, R0, #FIQ_Stack_Size

;  Enter IRQ Mode and set its Stack Pointer
                MSR     CPSR_c, #Mode_IRQ:OR:I_Bit:OR:F_Bit
                MOV     SP, R0
                SUB     R0, R0, #IRQ_Stack_Size

;  Enter Supervisor Mode and set its Stack Pointer
                MSR     CPSR_c, #Mode_SVC:OR:F_Bit
                MOV     SP, R0
                SUB     R0, R0, #SVC_Stack_Size
			B 	START

;----------------------------DO NOT CHANGE ABOVE THIS COMMENT--------------------------------
;--------------------------------------------------------------------------------------------
;--------------------------------------------------------------------------------------------

; Constant Data used in Counting Loop
Addr_SCS		DCD		0xE01FC1A0
MASK			DCD		0xFEFEFEFE
ISOLATE1		DCD		0x00010001
ISOLATE2		DCD		0x01000100
ISOLATE3		DCD		0x0000FFFF
ISOLATE4		DCD		0xFFFF0000
							
START		  	LDR R1, Addr_SCS			; Address of the System Control and Status (SCS) Flags
				MOV R2, #1								
				STR R2, [R1]			; Set the SCS to 1 to allow Fast Input
				
				LDR R1, Addr_FIOMASK		; Address of the Input Mask
				LDR R2, MASK							
				STR R2, [R1]			; Set the Mask to FEFEFEFE
		
				LDR R2, Addr_FIOPIN		; Address of the input pins
				LDR R11, [R2]

				LDR R8, ISOLATE1		; Load R8 with 00010001
				LDR R9, ISOLATE2		; Load R9 with 01000100					
								
LOOP			LDR R12, [R2]				; Load Input Pins				
				EOR R3, R11, R12		; Exclusive OR the previous value to the next to determine changes
				
				AND R6, R3, R8			; And Result of XOR with 00010001
				ADD R4, R4, R6			; Store Addition of 1st and 3rd pin in R4
				
				AND R7, R3, R9			; And Result of XOR with 01000100
				LSR R7, R7, #8			; Shift by 8 to the right to prevent overflows
				ADD R5, R5, R7			; Store Addition of 2nd and 4th pin in R5
				
				MOV R11, R12			; Store Value of input pins for comparison in the next iteration
				TST R0, #1			; AND R0 with 1
				BEQ 	LOOP			; Branch to LOOP if interrupt has happened									

FINISH			LDR R3, ACBASE				; Load address of ACBASE
				LDR R6, ISOLATE3		; Load R6 with 0000FFFF
				LDR R7, ISOLATE4		; Load R7 with FFFF0000
					
				MOV R8, R4								
				AND R8, R8, R6			; And the addition of the 1st and 3rd pin with 0000FFFF to get result of 1st pin
				LSR R8, #1			; Divide by 2	
				STR R8, [R3]			; Store result in the Memory location
				
				MOV R9, R5
				AND R9, R9, R6			; And the addition of the 2nd and 4th pin with 0000FFFF to get result of 2nd pin
				LSR R9, #1			; Divide by 2
				STR R9, [R3, #4]		; Store result in the Memory location
				
				MOV R10, R4							
				AND R10, R10, R7		; And the addition of the 1st and 3rd pin with FFFF0000 to get result of 3rd pin
				LSR R10, #17			; Shift right by 16 bits and divide by 2
				STR R10, [R3, #8]		; Store result in Memory location
				
				MOV R11, R5
				AND R11, R11, R7		; And the addition of the 2nd and 4th pin with FFFF0000 to get result of 4th pin
				LSR R11, #17			; Shift right by 16 bits and divide by 2
				STR R11, [R3, #12]		; Store result in Memory location
				B LOOP_END
				
ISR_FUNC		MOV R0, #1				; Change R0 to 1
				SUBS R15, R14, #4		; Branch to Instruction before interrupt happened by writing to PC	
								
;--------------------------------------------------------------------------------------------
; PARAMETERS TO CONTROL SIMULATION, VALUES MAY BE CHANGED TO IMPLEMENT DIFFERENT TESTS
;--------------------------------------------------------------------------------------------
SIMCONTROL
SIM_TIME 		DCD  	100000	  			; length of simulation in cycles (100MHz clock)
P0_PERIOD		DCD   	28        			; bit 0 input period in cycles
P1_PERIOD		DCD   	28				; bit 8 input period in cycles
P2_PERIOD		DCD  	28				; bit 16 input period in cycles
P3_PERIOD		DCD	28				; bit 24 input period in cycles
;---------------------DO NOT CHANGE AFTER THIS COMMENT---------------------------------------
;--------------------------------------------------------------------------------------------
;--------------------------------------------------------------------------------------------
LOOP_END		MOV R0, #0x7f00
				LDR R0, [R0] 			; read memory location 7f00 to stop simulation
STOP			B 	STOP
;-----------------------------------------------------------------------------
 				AREA	DATA, READWRITE

P0COUNT			DCD		0
P1COUNT			DCD		0
P2COUNT			DCD		0
P3COUNT			DCD		0
;------------------------------------------------------------------------------			
                AREA    STACK, NOINIT, READWRITE, ALIGN=3

Stack_Mem       SPACE   USR_Stack_Size
__initial_sp    SPACE   ISR_Stack_Size

Stack_Top


        		END                     ; Mark end of file

