; Asbestos Garage Roof Cost Calculator
; NASM x86-64 Assembly for Linux
; 
; This program uses the following assumptions (from the blog):
;   Garage Types:
;     1) Single Garage (3m x 6m):   Area = 18 m², Removal Cost = £800, Additional Cost = £750
;     2) Double Garage (5.5m x 6m): Area = 33 m², Removal Cost = £1200, Additional Cost = £1000
;     3) Three-Car Garage (9m x 6m): Area = 54 m², Removal Cost = £1800, Additional Cost = £1250
;
;   Roofing Materials (total cost per m², using mid-range averages):
;     1) Steel:            £60 per m²
;     2) EPDM Rubber:      £85 per m²
;     3) GRP Fibreglass:   £118 per m²
;     4) Onduline Bitumen:  £30 per m²
;     5) Concrete Fibre:   £68 per m²
;
; Total Cost = Removal Cost + (Area * Material Cost per m²) + Additional Cost
;
; Assemble with: nasm -f elf64 calculator.asm && ld -o calculator calculator.o
; Run with: ./calculator

section .data
    prompt_garage db "Select Garage Type:", 10, _
                     "(1) Single Garage (18 m^2)", 10, _
                     "(2) Double Garage (33 m^2)", 10, _
                     "(3) Three-Car Garage (54 m^2)", 10, _
                     "Enter choice (1-3): ", 0
    prompt_garage_len equ $ - prompt_garage

    prompt_material db "Select Roofing Material:", 10, _
                       "(1) Steel", 10, _
                       "(2) EPDM Rubber", 10, _
                       "(3) GRP Fibreglass", 10, _
                       "(4) Onduline Bitumen", 10, _
                       "(5) Concrete Fibre", 10, _
                       "Enter choice (1-5): ", 0
    prompt_material_len equ $ - prompt_material

    result_msg db 10, "Total Estimated Cost: £", 0
    result_msg_len equ $ - result_msg

    newline db 10, 0

section .bss
    input_buf resb 16      ; buffer for user input
    num_str   resb 32      ; buffer for number-to-string conversion

section .text
    global _start

_start:
    ; --- Prompt for Garage Type ---
    mov     rax, 1                ; sys_write
    mov     rdi, 1                ; stdout
    mov     rsi, prompt_garage
    mov     rdx, prompt_garage_len
    syscall

    ; Read garage type choice
    call    read_int              ; result in rax (choice)
    mov     rbx, rax              ; save garage choice in rbx

    ; Based on choice, set area, removal cost, additional cost.
    ; We'll use:
    ;   r12 = area, r13 = removal cost, r14 = additional cost
    cmp     rbx, 1
    je      single_garage
    cmp     rbx, 2
    je      double_garage
    cmp     rbx, 3
    je      three_car
    ; If invalid, default to single garage:
single_garage:
    mov     r12, 18               ; area = 18 m²
    mov     r13, 800              ; removal cost = £800
    mov     r14, 750              ; additional cost = £750
    jmp     garage_type_done

double_garage:
    mov     r12, 33               ; area = 33 m²
    mov     r13, 1200             ; removal cost = £1200
    mov     r14, 1000             ; additional cost = £1000
    jmp     garage_type_done

three_car:
    mov     r12, 54               ; area = 54 m²
    mov     r13, 1800             ; removal cost = £1800
    mov     r14, 1250             ; additional cost = £1250

garage_type_done:
    ; --- Prompt for Roofing Material ---
    mov     rax, 1                ; sys_write
    mov     rdi, 1                ; stdout
    mov     rsi, prompt_material
    mov     rdx, prompt_material_len
    syscall

    ; Read material choice
    call    read_int              ; result in rax (choice)
    mov     rbx, rax              ; store material choice in rbx

    ; We'll use r15 for cost per m².
    cmp     rbx, 1
    je      material_steel
    cmp     rbx, 2
    je      material_epdm
    cmp     rbx, 3
    je      material_grp
    cmp     rbx, 4
    je      material_onduline
    cmp     rbx, 5
    je      material_concrete
    ; Default to steel if invalid:
material_steel:
    mov     r15, 60               ; Steel = £60 per m²
    jmp     material_done

material_epdm:
    mov     r15, 85               ; EPDM Rubber = £85 per m²
    jmp     material_done

material_grp:
    mov     r15, 118              ; GRP Fibreglass = £118 per m²
    jmp     material_done

material_onduline:
    mov     r15, 30               ; Onduline Bitumen = £30 per m²
    jmp     material_done

material_concrete:
    mov     r15, 68               ; Concrete Fibre = £68 per m²

material_done:
    ; --- Calculate Total Cost ---
    ; installation_cost = area * material_cost_per_m²
    mov     rax, r12            ; rax = area
    imul    rax, r15            ; rax = area * cost_per_m²

    ; Add removal cost and additional cost:
    add     rax, r13            ; add removal cost
    add     rax, r14            ; add additional cost
    ; Now rax holds the total estimated cost.

    ; --- Convert rax (total cost) to a string ---
    mov     rdi, num_str        ; destination buffer
    call    itoa                ; converts number in rax to string in num_str

    ; --- Print the result ---
    ; Print result message
    mov     rax, 1
    mov     rdi, 1
    mov     rsi, result_msg
    mov     rdx, result_msg_len
    syscall

    ; Print the number string (which is now in num_str)
    mov     rax, 1
    mov     rdi, 1
    mov     rsi, num_str
    ; Determine length of num_str by scanning for zero
    mov     rcx, 0
find_end:
    cmp     byte [num_str + rcx], 0
    je      print_num
    inc     rcx
    jmp     find_end
print_num:
    mov     rdx, rcx
    syscall

    ; Print a newline
    mov     rax, 1
    mov     rdi, 1
    mov     rsi, newline
    mov     rdx, 1
    syscall

    ; --- Exit ---
    mov     rax, 60            ; sys_exit
    xor     rdi, rdi
    syscall

; ---------------------------------------------------------
; read_int: Reads a line from stdin, converts it from ASCII to integer.
; Returns the integer in rax.
; Clobbers rax, rbx, rcx, rdx, rsi.
read_int:
    ; Clear input buffer
    mov     rdi, input_buf
    mov     rcx, 16
    xor     rax, rax
    rep stosb

    ; Read from stdin
    mov     rax, 0            ; sys_read
    mov     rdi, 0            ; stdin
    mov     rsi, input_buf
    mov     rdx, 15
    syscall

    ; Convert ASCII digits to integer (assumes valid positive number)
    xor     rax, rax          ; result = 0
    mov     rsi, input_buf
atoi_loop:
    mov     bl, byte [rsi]
    cmp     bl, 10            ; newline? (LF)
    je      atoi_done
    cmp     bl, 0             ; null terminator
    je      atoi_done
    cmp     bl, '0'
    jb      atoi_done       ; if not a digit, stop
    cmp     bl, '9'
    ja      atoi_done
    ; rax = rax * 10 + (bl - '0')
    imul    rax, rax, 10
    sub     bl, '0'
    add     rax, rbx
    inc     rsi
    jmp     atoi_loop
atoi_done:
    ret

; ---------------------------------------------------------
; itoa: Converts the unsigned integer in rax to a null-terminated string.
; Expects destination pointer in rdi (buffer must be large enough).
; Uses rax as the input number; the converted string will be placed in [rdi].
; (This routine writes the digits in reverse order into a temporary buffer,
; then reverses them into the destination.)
itoa:
    push    rbx
    push    rcx
    push    rdx
    ; If number is zero, handle explicitly.
    cmp     rax, 0
    jne     itoa_convert
    mov     byte [rdi], '0'
    mov     byte [rdi+1], 0
    jmp     itoa_done

itoa_convert:
    ; Use rax as the number; store digits in temporary buffer on the stack.
    sub     rsp, 32            ; allocate temporary space
    mov     rsi, rsp           ; rsi points to temp buffer
    xor     rcx, rcx           ; digit count = 0

itoa_loop_conv:
    xor     rdx, rdx
    mov     rbx, 10
    div     rbx                ; divide rax by 10; quotient in rax, remainder in rdx
    add     rdx, '0'
    mov     byte [rsi + rcx], dl
    inc     rcx
    cmp     rax, 0
    jne     itoa_loop_conv

    ; Now reverse the digits into destination buffer in rdi.
    mov     rbx, rcx           ; rbx = digit count
    mov     rdx, rdi           ; rdx = dest pointer
itoa_reverse_loop:
    dec     rbx
    mov     al, [rsi + rbx]
    mov     [rdx], al
    inc     rdx
    cmp     rbx, 0
    jne     itoa_reverse_loop

    ; Terminate string
    mov     byte [rdx], 0

    add     rsp, 32            ; free temporary space
itoa_done:
    pop     rdx
    pop     rcx
    pop     rbx
    ret
