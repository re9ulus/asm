### syntax
`b` byte      ( 8bit)
`w` word      (16bit)
`l` long word (32bit)
`q` quad word (64bit)

constant starts with `$`

address: `disp(base, index, scale)`
addr = base + index * scale + disp
where base, index - registers, disp - const, scale - 1,2,4,8

### mov
```
; write 8bit constans to AL
movb $0xff, %al

; write 8bit constant to R8L
movb $0xff, %r8b

; write 16bit constant to BX
movw $0xffff, %bx

; write 32bit constant to EDI
movl $0xffffffff, %edi

; write 64bit constant to R15
movq $0xffffffffffffffff, %r15

; copy AL into BH
movb %al, %bh

; write AL by addr 0x12345
movb %al, 0x12345

; write 8bit const by addr in register %rsp
movb $0xff, (%rsp)

; write value of bx by addr in (%rax + 10)
movw %bx, 10(%rax)
```

### movsx
### movzx

### add
```
; Sum 5 and EDI, put result into EDI
addl $5, %edi

; Sum R8 and R10
addq %r8, %r10

; Sub BH and value by addr 0x10, put result by addr 0x10
addb %bh, 0x10
```

### shift
```
; move EDI by 1 to left
shll $1, %edi
```

### eflags
CF -
ZF -
SF -
OF -
PF -
AF -
