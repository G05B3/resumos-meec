# Laboratório 2

## Exercicio de casa

###  by RodsCoimbra

::: details Resolução

```asm6502
.data
a: .word 0
b: .word 0
c: .word 0
d: .word 0
y: .word 0

    .text
rede_neuronal_xor:
    la a1, c
    la a2, d
    la a3, y
    lw s0, a
    lw s1, b
    li s2, 2
    li s3, -2
    jal x1, neuronio
    sw x10, 0(a1)
    li s2, -2
    li s3, 2
    jal x1, neuronio
    sw x10, 0(a2)
    lw s0, 0(a1)
    lw s1, 0(a2)
    li s2, 2
    li s3, 2
    jal x1, neuronio
    sw x10, 0(a3)


    li x17, 1
    ecall
    li x17, 10
    ecall


neuronio:
    addi x2, x2, -12
    sw x1, 8(x2)
    sw s2, 4(x2)
    sw s0, 0(x2)
    jal x1, multiplica
    lw s5, 0(x2)
    addi x2,x2, 4
    sw s3, 4(x2)
    sw s1, 0(x2)
    jal x1, multiplica
    lw s6, 0(x2)
    addi x2, x2, 4
    add s6, s6, s5
    addi s6, s6, -1
    lw x1, 8(x2)
    addi x2, x2, 12
    bge s6, x0, retorno
    li x10, 0
    jalr x0, x1, 0
retorno:
    li x10, 1
    jalr x0, x1, 0

multiplica:
    lw t0, 0(x2)
    lw t1, 4(x2)
    li t2, 0
    bgtz t1, for
    neg t1,t1
    neg t0, t0

for: 
    add t2, t0, t2
    addi t1, t1, -1
    bgtz t1, for

    addi x2, x2, -4
    sw t2, 0(x2)
    jalr x0, x1, 0
```

:::

### by Tomás Martins

::: details Resolução

```asm6502
.data
a: .word 0
b: .word 0
.text
rede_neuronal_xor:
	lw a1,a
	lw a2,b
	li a3,-2
	li a4,2
	jal x3,neuronio
	mv s1,a0
	li a3,2
	li a4,-2
	jal x3,neuronio
	mv s2,a0
	mv a1,s1
	mv a2,s2
	li a3,2
	li,a4,2
	jal x3,neuronio
	
	li x17,1
	ecall
	
	li x17,10
	ecall

neuronio:
	mv a5,a1
	mv a6,a4
	jal x1,multiplica
	lw s3,0(sp)
	addi sp,sp,4
	mv a5,a2
	mv a6,a3
	jal x1,multiplica
	lw s4,0(sp)
	addi sp,sp,4
	li a5,-1
	add s3,s3,s4
	add s3,s3,a5
	bgez s3,if1
	li a0,0
	jalr x0,x3,0
if1:
	li a0,1	
	jalr x0,x3,0

multiplica:
	addi sp,sp,-12
	mv t1,a5
	sw t1,4(sp)
	mv t2,a6
	sw t2,0(sp)
	li t3,0
	bgez t2,loop
loop1:
	neg t2,t2
	add t3,t3,t1
	addi t2,t2,-1
	bgtz t2,loop1
	neg t3,t3
	sw t3,8(sp)
	addi sp,sp,8
	ret
loop:
	add t3,t3,t1
	addi t2,t2,-1
	bgtz t2,loop
	sw t3,8(sp)
	addi sp,sp,8
	ret
```

:::

## Lab 2 v1 - by Martim Rendeiro Bento and João Barreiros C. Rodrigues

Modifique o código de forma a implementar a função: $y = A \oplus \overline{C}$.
A função **y** deve receber os 3 parametros pela pilha.
As funções OR e A $\oplus$ $\overline{C}$ podem ser implementados por neurónios com os seguintes pesos:

- OR: w1 = 2, w2 = 2, b= -1
- A $\oplus$ $\overline{C}$: w1 = 2, w2 = -2, b= -1

Teste a função utilizando código que a chama 8 vezes com os valores da tabela de verdade (não é necessário um loop) e apresente os resultados na consola.

::: details Resolução

```asm6502
	.data

a:	.word 0
b:	.word 0
c:	.word 1

	.text
#x14 = w1; x15 = w2; x16 = b; x3 =


rede_neuronal_xor:
lw x3, a
lw x4, b
li x14, 2
li x15, -2
li x16, -1
jal neuronio
mv x20, x19
li x14, -2
li x15, 2
li x16, -1
jal neuronio
li x14, 2
li x15, 2
li x16, -1
mv x3, x20
mv x4, x19
jal neuronio  #daqui sai A xor B no x19
mv x20, x19 #gravar A xor B no x20
li x14, 3
li x15, -2
li x16, -2
lw x3, c
lw x4, a
jal neuronio #daqui sai C and (Not A)
mv x21, x19 #gravar C and (Not A) no x21
li x14, 2
li x15, 2
li x16, -1
mv x3, x20
mv x4, x21
jal neuronio #daqui deve sair (A xor B) or (C and (Not A))
jal x0, end

neuronio:
addi sp, sp, -4
sw x1, 0(sp)

mv x10, x3
mv x11, x14
jal multiplica
lw x18, 0(sp)
addi sp, sp, 4
mv x10, x4
mv x11, x15
jal multiplica
lw x19, 0(sp)
addi sp, sp, 4
add x19, x19, x18
add x19, x19, x16
lw x1, 0(sp)
addi sp, sp, 4
blt x19, zero, null
li x19, 1
jalr x0, x1, 0
null:
li x19, 0
jalr x0, x1, 0

multiplica:
li x12, 0
li x13, 0
blt x11, zero, negative
while:
add x13, x13, x10
addi x12, x12, 1
blt x12, x11, while
addi sp, sp, -4
sw x13, 0(sp)
jalr x0, x1, 0
negative:
sub x13, x13, x10
addi x12, x12, -1
bge x12, x11, negative
addi sp, sp, -4
sw x13, 0(sp)
jalr x0, x1, 0

end:
mv x10, x19
li x17, 1
ecall
li x17, 10
ecall

```

:::
## Lab 2 v2 - by João Barreiros C. Rodrigues and Martim Rendeiro Bento 
::: details Resolução

```asm6502
#--------------------------------------------------------------------------------------------------+
# Laboratory 2|Calculate the truth table of the expression (a XOR b) and (a or c).                 |
#             |                                                                                    |
#--------------------------------------------------------------------------------------------------+
# Author: Joao Barreiros C. Rodrigues (Joao-Ex-Machina), Martim Rendeiro Bento (G05B3)             |
# Date: 14 April 2021                                                                              |
#-------------------------------------------------------------------------------------------------*/
    .data

a:    .word 0 #LSB
b:    .word 0 
c:    .word 0 #MSB

    .text
li x24, 1 #initialize counter register
cicle:
li x25, 9
la x26, a #initialize variable pointer registers
la x27, b
la, x5, c

li x28, 0
addi x28, x24, -5
blt x28, zero, czero #for the first four numbers MSB=0
li x28, 1
sw x28, 0(x5)
czero:
li x28, 2
rem x28, x24, x28 #for all even numbers the LSB = 0 
xori x28, x28, 1 #adjust a since 000 is taken as the the first cicle and not cicle zero
sw x28, 0(x26)

#B=1 from 010 (3rd cicle) until it 100 (5th cicle).
li x28, 3 
bne x28, x24, not3
li x28, 1
sw x28, 0(x27)
not3:
li x28, 5
bne x28, x24, not5
li x28, 0
sw x28, 0(x27)
not5:
#B=1 from 110 (7th cicle) until the end
li x28, 7
bne x28, x24, not7
li x28, 1
sw x28, 0(x27)
not7:
addi x24, x24, 1
jal x0, rede_neuronal_xor



rede_neuronal_xor:
lw x3, a
lw x4, b
li x14, 2
li x15, -2
li x16, -1
jal neuronio
mv x20, x19
li x14, -2
li x15, 2
li x16, -1
jal neuronio
li x14, 2
li x15, 2
li x16, -1
mv x3, x20
mv x4, x19
jal neuronio #a xor b
mv x23, x19
li x14, 2
li x15, 2
li x16, -1
lw x3, a
lw x4, c
jal neuronio #a or c
li x14, 1
li x15, 1
li x16, -2
mv x3, x23
mv x4, x19
jal neuronio # (a xor b) and (a or c)
jal x0, end

neuronio:
mv x22, x1 #save intial ra
addi sp, sp, -8
sw x3, 4(sp)
sw x14, 0(sp)
jal multiplica
lw x18, 0(sp)
addi sp, sp, 4
addi sp, sp, -8
sw x4, 4(sp)
sw x15, 0(sp)
jal multiplica
lw x19, 0(sp)
addi sp, sp, 4
mv x12, x16
add x19, x19, x18
add x19, x19, x16
mv x1, x22 #reload inital ra
blt x19, zero, null
li x19, 1
jalr x0, x1, 0
null:
li x19, 0
jalr x0, x1, 0

multiplica:
li x12, 0 
li x13, 0
lw x11, 0(sp)
lw x10, 4(sp)
addi sp sp 8
blt x11, zero, negative
while:
add x13, x13, x10
addi x12, x12, 1
blt x12, x11, while
addi sp, sp, -4
sw x13, 0(sp)
jalr x0, x1, 0
negative:
sub x13, x13, x10
addi x12, x12, -1
bge x12, x11, negative
addi sp, sp, -4
sw x13, 0(sp)
jalr x0, x1, 0

end:
mv x10, x19
li x17, 1
ecall
blt x24, x25, cicle #while x24 < 9 restart
li x17, 10
ecall

```
