# Notes personnelles pour le challenge :

On remarque assez vite les symboles des fonctions importantes à regarder : les fonctions `check`, et les `check_char_*`.

Analysons-le code assembleur : 

## Analyse de <check_char_0>

```
//Stack frame setup
1	: push 		%ebp 							 
2 	: mov 		%esp, %ebp 						 
3 	: push 		%ebx 							
4 	: sub 		$0x4,%esp 						
5 	: call 		8049e74 <__x86.get_pc_thunk.ax>
6 	: add 		$0x91469,%eax 					


7 	: mov 		0x8(%ebp),%eax 					> %eax <- %ebp + 8
8 	: mov 		%al,-0x8(%ebp) 					> %ebp - 8 <- %al 
9 	: movzbl 	-0x8(%ebp),%ebx		 			> %ebx <- %ebp - 8 = %al zero extended
10 	: sub 		$0x3f,%ebx 					> %ebx <- %ebx - 63
11 	: sub 		$0x74,%ebx 					> %ebx <- %ebx - 116
12 	: mov 		$0x4e,%edx 					> %edx <- 78
13 	: mov 		%ebx,%eax 					> %eax <- ebx = %al - 63 - 116 
14 	: imul 		%edx,%eax 					> %eax <- %eax * %edx = (%al - 63 - 116) * 78
15 	: mov 		%eax,%ebx 					> %ebx <- %eax 
16 	: cmp 		$0xe0,%bl 						
17 	: je 		8049bc4 <check_char_0+0x39> 			> si %bl == 224 on renvoie 1
18 	: mov 		$0x0,%eax					> sinon on renvoie 0
19 	: jmp 		8049bc9 <check_char_0+0x3e>		
20 	: mov 		$0x1,%eax 						
[//Sortie de fonction]
```

On pourrait voir la fonction comme (?) :
```C
int check_char_0 (char a) {
	char eax = a;
	char ebx = (eax - 63 - 116) * 78;
	if (ebx%256 == 224) { 				
		return 1;
	}
	else {
		return 0;
	}
}
```
==> On veut `ebx%256 == 224`,

donc `(a - 63 - 116) * 78 %256 == 224`,

avec `a = 'C'`, ça devrait marcher !



## Analyse de <check_char_1>

```
//Stack frame setup
1 	: push 		%ebp
2 	: mov    	%esp,%ebp
3 	: push   	%ebx
4 	: sub    	$0x4,%esp
5 	: call   	8049e74 <__x86.get_pc_thunk.ax>
6 	: add    	$0x91425,%eax


7 	: mov    	0x8(%ebp),%eax 					> %eax <- %ebp + 8
8 	: mov    	%al,-0x8(%ebp)					> %ebp - 8 <- %al
9 	: movzbl 	-0x8(%ebp),%ebx 				> %ebx <- %ebp - 8 == %al
10 	: shl    	$0x5,%ebx 						> %ebx << 5 // = %ebx * 32
11 	: and    	$0xffffffef,%ebx				> %ebx <- %ebx & 0xffffffef
12 	: test   	%bl,%bl 						> Si %bl == 0
13 	: je     	8049bfb <check_char_1+0x2c> 	> On renvoie 1
14 	: mov    	$0x0,%eax 						> Sinon on renvoie 0
15 	: jmp    	8049c00 <check_char_1+0x31> 
16 	: mov    	$0x1,%eax 						
[//Sortie de fonction]
```

On peut voir la fonction comme (?) :

```C
int check_char_1 (char a) {
	char eax = a;
	char ebx = (eax << 5) & 0ffffffef;
	if (ebx%256 == 0) {
		return 1;
	}
	else {
		return 0;
	}
}
```
on veut `bl == 0`,

donc `ebx % 256== 0`;

donc `(a * 32) & 0xffffffef % 256 == 0`

avec `a = 'H'` ou `a = 'P'` ou `a = 'X'` ou <code>a = '\`'</code> ou `a = 'h'` ou `a = 'p'` ou `a = 'x'`, ça devrait marcher !

## Analyse de <check_char_2>

```
//Stack frame setup
1	: push 	 	%ebp
2 	: mov    	%esp,%ebp
3 	: push   	%ebx
4 	: sub    	$0x4,%esp
5 	: call   	8049e74 <__x86.get_pc_thunk.ax>
6 	: add    	$0x913ee,%eax


7 	: mov    	0x8(%ebp),%eax 					> %eax <- %ebp + 8
8 	: mov    	%al,-0x8(%ebp) 					> %ebp - 8 <- %al
9 	: movzbl	-0x8(%ebp),%ebx 				> %ebx <- %ebp - 8 = %al
10 	: movzbl 	%bl,%eax						> %eax <- %bl zero extended
11 	: push   	$0x82 							> on empile 0x82
12 	: push   	%eax							> on empile %eax
13 	: call   	8049b35 <r>						> on appelle <r> sur %eax et %0x82
14 	: add    	$0x8,%esp						> %esp <- %esp + 8 	== 0x82
15 	: mov    	%eax,%ebx 						> %ebx < %eax = %bl = %al zero extended
16 	: cmp    	$0xcc,%bl 						> %bl == 0xcc ?
17 	: je     	8049c40 <check_char_2+0x3a> 	> si oui on saute ligne 20
18 	: mov    	$0x0,%eax 						> si non on retourne 0
19 	: jmp    	8049c45 <check_char_2+0x3f>		> puis on sort
20 	: mov    	$0x1,%eax 						> on retourne 1
21  : mov   	-0x4(%ebp),%ebx
22 	: leave  
23 	: ret    
```

On peut voir la fonction comme :
```C
int check_char_2(char a) {
	char eax = a;
	char ebx = a % 256;
	eax = r(ebx,0x82);
	ebx = eax;
	if (ebx%256 == 0xcc) {
		return 1;
	} 
	else {
		return 0;
	}
}
```
On veut donc ebx % 256 == 204.

On a ebx = r(a%256, 130);

donc ebx = (a << 8 - 130&7) lor (a >> 130&7); 			//Cf ci-dessous

donc ebx = (a << 8 - 2) lor (a >> 2);

donc ebx = (a * 64) lor (a / 4);

Avec a = '3', ça peut marcher ! 


### Analyse de \<r> nécessaire

```
//Stack frame setup
1 	: push   	%ebp
2 	: mov    	%esp,%ebp
3 	: push   	%ebx
4 	: sub    	$0x18,%esp 						> %esp <- %esp - 24
5 	: call   	8049e74 <__x86.get_pc_thunk.ax>
6 	: add    	$0x914bf,%eax

7 	: mov    	0x8(%ebp),%edx 					> %edx <- %ebp + 8					// param 1 = %eax du check_char_2
8 	: mov    	0xc(%ebp),%eax					> %eax <- %ebp + 12					// param 2 = 0x82
9 	: mov    	%dl,-0x18(%ebp) 				> %ebp - 24 <- %dl = x
10 	: mov    	%al,-0x1c(%ebp) 				> %ebp - 28 <- %al = y
11 	: movzbl 	-0x1c(%ebp),%eax 				> %eax <- %ebp - 28 = y
12 	: and    	$0x7,%eax 						> %eax = %eax & 7 = y&7
13 	: mov    	%al,-0x5(%ebp) 					> %ebp - 5 <- %al = y&7 
14 	: movzbl 	-0x18(%ebp),%edx 				> %edx <- %ebp - 24 = x
15 	: movzbl 	-0x5(%ebp),%eax 				> %eax <- %ebp - 5 = y&7
16 	: mov    	%eax,%ecx 						> %ecx <- %eax = y&7
17 	: sar    	%cl,%edx 						> on shifte %edx de %cl vers la droite, et on garde le signe, %edx = x >> y&7
18 	: mov    	%edx,%eax 						> %eax <- %edx = x >> y&7 
19 	: mov    	%eax,%ebx 						> %ebx <- %eax = x >> y&7
20 	: movzbl 	-0x18(%ebp),%edx 				> %edx <- %ebp-24, zero extended == x
21 	: movzbl 	-0x5(%ebp),%eax 				> %eax <- %ebp-5, zero extended  == y&7
22 	: mov    	$0x8,%ecx						> %ecx <- 8
23 	: sub    	%eax,%ecx						> %ecx <- %ecx - %eax = 8 - y&7
24 	: mov    	%ecx,%eax 						> %eax <- %ecx = 8 - y&7
25 	: mov    	%eax,%ecx 						> %ecx <- %eax = 8 - y&7
26 	: shl    	%cl,%edx 						> %edx <- %edx = x << 8 - y&7
27 	: mov    	%edx,%eax 						> %eax <- %edx = x << 8 - y&7
28 	: or     	%ebx,%eax 						> %eax <- x << 8 - y&7 ||  x >> y&7
29 	: add    	$0x18,%esp 						> %esp += 24
30 	: pop    	%ebx 							> %ebx = *(%esp); %esp +=4 
31 	: pop    	%ebp 							> %ebp = *(esp); %esp +=4
32 	: ret    
```

On peut voir la fonction comme :

```C
int r(int x, int y) {
	return (x << 8 - y&7) lor (x >> y&7);
}
```


## Analyse de <check_char_3>

```
//Stack frame setup
1	: push   %ebp
2	: mov    %esp,%ebp
3	: push   %ebx
4	: sub    $0x4,%esp
5	: call   8049e74 <__x86.get_pc_thunk.ax>
6	: add    $0x913aa,%eax

7	: mov    0x8(%ebp),%eax						> %eax <- %ebp + 8
8	: mov    %al,-0x8(%ebp)						> %ebp - 8 <- %al
9	: movzbl -0x8(%ebp),%ebx 					> %ebx <- %ebp - 8 = %al zero extended
10	: mov    $0x3e,%edx 						> %edx <- 0x3e
11	: mov    %ebx,%eax 							> %eax <- %ebx = %al zero extended
12	: imul   %edx,%eax 							> %eax <- %eax * %edx = %al * 62
13	: mov    %eax,%ebx 							> %ebx <- %eax = %al * 62
14	: xor    $0x65,%ebx 						> %ebx <- %ebx ^ 0x65 = %(al*62)^101
15	: cmp    $0x35,%bl 							> si %ebx % 256 == 53...
16	: je     8049c80 <check_char_3+0x36> 		> on renvoie 1 
17	: mov    $0x0,%eax							> sinon on renvoie 0
18	: jmp    8049c85 <check_char_3+0x3b>
19	: mov    $0x1,%eax
20	: add    $0x4,%esp
21	: pop    %ebx
22	: pop    %ebp
23	: ret  
```

On peut voir la fonction comme :

```C
int check_char_3(char a) {
	char eax = a;
	char ebx = a%256; 		// = a car a est un caractère ascii
	eax = a * 62;
	ebx = eax^0x65; 		// = (a*62)^101
	if (ebx %256 == 53){
		return 1;
	}
	else {
		return 0;
	}
}
```

On veut `ebx%256 == 53`,

donc on veut `(a*62)^101 %256 == 53`,

avec `a = 'X'`, ça peut marcher !

## Analyse de <check_char_4>

```
\\\Stack frame setup
1 	: push   %ebp
2 	: mov    %esp,%ebp
3 	: push   %ebx
4 	: sub    $0x4,%esp
5 	: call   8049e74 <__x86.get_pc_thunk.ax>
6 	: add    $0x91369,%eax

7 	: mov    0x8(%ebp),%eax 				> %eax <- %ebp + 8
8	: mov    %al,-0x8(%ebp)	 				> %ebp - 8 <- %al = %eax % 256
9	: movzbl -0x8(%ebp),%ebx	 			> %ebx <- %ebp - 8 = %eax
10	: mov    $0x74,%edx 					> %edx <- 0x74
11	: mov    %ebx,%eax 						> %eax <- %ebx
12	: imul   %edx,%eax						> %eax <- %eax * %edx = %eax * 0x74
13	: mov    %eax,%ebx 						> %ebx <- %eax
14	: not    %ebx 							> %ebx <- !%ebx
15	: cmp    $0xbf,%bl 						> Si %bl == 0xbf...
16	: je     8049cc0 <check_char_4+0x35 	> On renvoie 1
17	: mov    $0x0,%eax						> Sinon on renvoie 0
18	: jmp    8049cc5 <check_char_4+0x3a>
19	: mov    $0x1,%eax
20	: add    $0x4,%esp
21	: pop    %ebx
22	: pop    %ebp
23	: ret    
```

On peut voir la fonction comme :
```C
int check_char_4(char a){
	char eax = a;
	char ebx = ~(eax*116);
	if (ebx%256 == 191) {
		return 1;
	}
	else {
		return 0;
	}
}
```

On veut `ebx%256 == 191`,

donc `!(a * 116)%256 == 191`,

donc `(a * 116)%256 == 64`,

avec `a = 'P'`, ça devrait marcher !


## Analyse de <check_char_5>


```
//Setup de la stack frame
1 	: push   %ebp
2	: mov    %esp,%ebp
3	: push   %ebx
4	: sub    $0x4,%esp
5	: call   8049e74 <__x86.get_pc_thunk.ax>
6	: add    $0x91329,%eax

7	: mov    0x8(%ebp),%eax 					> %eax <- %ebp + 8
8	: mov    %al,-0x8(%ebp)						> %ebp - 8 <- %al = %eax%256
9	: movzbl -0x8(%ebp),%ebx					> %ebx <- %ebp - 8 = %al
10	: xor    $0x3b,%ebx 						> %ebx <- %ebx^0x3b
11	: cmp    $0x50,%bl 							> si %bl == 0x50..
12	: je     8049cf5 <check_char_5+0x2a>		> on renvoie 1
13	: mov    $0x0,%eax 							> sinon on renvoie 0
14	: jmp    8049cfa <check_char_5+0x2f>
15	: mov    $0x1,%eax
16	: add    $0x4,%esp
17	: pop    %ebx
18	: pop    %ebp
19	: ret    
```

On peut voir la fonction comme :
```C
int check_char_5(char a) {
	char eax = a;
	char ebx = a^0x3b;
	if (ebx%256 == 80) {
		return 1;
	}
	else {
		return 0;
	}
}
```

On veut `ebx%256 == 80`,

donc `a^59 == 80`,

avec `a = 'k'`, ça peut marcher ! 


## Analyse de \<check>

```
//stack setup
0	: push 		%ebp
1 	: mov 		%esp,%ebp
2 	: call 		8049e74 <__x86.get_pc_thunk.ax>
3 	: add 		$0x912f8,%eax 					

//On récupère le premier caractère
4 	: mov 		0x8(%ebp),%eax						
5 	: movzbl 	(%eax),%eax 					 
6 	: movsbl 	%al,%eax							
7 	: push 		%eax 							
8 	: call 		8049b8b <check_char_0> 			
9 	: add 		$0x4,%esp 						
10 	: test 		%eax,%eax 						> si %eax == 0...
11 	: je 		8049dab 						> On saute à la ligne 59 (ie on fail)

//On récupère le deuxième caractère
12 	: mov 		0x8(%ebp),%eax 					
13 	: add 		$0x1,%eax 						
14 	: movzbl 	(%eax),%eax
15 	: movsbl 	%al,%eax							
16 	: push 		%eax 					  
17 	: call 		8049bcf <check_char_1> 

[...]

53 	: call 		8049ccb <check_char_5> 
54 	: add 		$0x4,%esp 						
55 	: test 		%eax,%eax 						> Si %eax == 0...
56 	: je 		8049dab 						> On saute à la ligne 59 (ie on fail)
57 	: mov 		$0x1,%eax 						> On met 1 dans %eax
58 	: mov 		8049db0 <check+0xb0> 			> On sort de la fonction
59 	: mov 		$0x0,%eax 						> On met 0 dans %eax
60	: leave 									
61 	: ret 										
```

On pourrait voir la fonction comme :
```C
int check(char* chaine) {
	if (chaine[0] == 'C') { 			
		if (chaine[1] == 'H' || chaine[1] == 'P' || chaine[1] == 'X' || chaine[1] == '`' || chaine[1] == 'h' || chaine[1] == 'p' || chaine[1] == 'x') {
			if (chaine[2] == '3') {
				if (chaine[3] == 'X') {
					if (chaine[4] == 'P') {
						if (chaine[5] == 'k') {
							return 1;
						}
					}
				}
			}
		}
	}
	else {
		return 0;
	}
}
```

## Résultats

On obtient bien un "Success !" avec les chaînes :
 - CH3XPk
 - CP3XPk
 - CX3XPk
 - C\`3XPk
 - Ch3XPk
 - Cp3XPk
 - Cx3XPk

## Remarques personnelles

 - J'ai utilisé objdump, avec une syntaxe AT&T.
 - J'avais retiré les adresses pour les remplacer par des numérotations commençant à 1 pour que ça soit plus lisible.
 - J'ai passé du temps à décortiquer `<main>` et `<_start>`, ce qui n'a pas été utile.
 - J'ai utilisé un tableur pour résoudre les équations finales.
 - J'avais fait une erreur de frappe en écrivant 0x6b à la place de 0x3b dans <check_char_5>, ce qui a été très dur à voir car je n'avais aucun moyen de m'en rendre compte rapidement.
 - Beaucoup de café a été bu.  