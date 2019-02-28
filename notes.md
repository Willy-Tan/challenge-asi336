Notes personnelles pour le challenge :

Utilisé avec objdump, syntaxe AT&T

Registres et rôles généraux (peut ne pas servir à ça) :
- %ebp : base pointer, pointe à la base de la pile
- %esp : stack pointer, pointe à l'endroit courant de la pile
- %eax : accumulateur pour les opérations arithmétiques, valeur de retour
- %ebx : valeur non-volatile
- %ecx : compteur de boucle
- %edx : variable locale, paramètre de fonction
- %esi : indice de source, pour les string généralement

##Analyse de <\_start> (inutile ?)

```
//Stack frame setup
0 	: xor 		%ebp, %ebp						> on remet %ebp à 0
1 	: pop 		%esi 							> ??
2 	: mov 		%esp, %ecx 						> on met le contenu de %esp dans %ecx >> Remonter jusqu'à %esp 
3 	: and 		$0xfffffff0,%esp				> on retire les 4 derniers bits de %esp
4 	: push 		%eax							> on met %eax dans la pile
5 	: push 		%esp 							> on met %esp dans la pile
6 	: push  	%edx 							> on met %edx dans la pile
7 	: call 		80499f3 <_start+0x33>			> on va à la ligne 19 et on stock dans ret l'instruction 8
8 	: add 		$0x91630,%ebx 					> on additionne 0x91630 à %ebx >> contenu de ebx ?
9 	: lea 		-0x90470(%ebx),%eax				> chargement de l'adresse %ebx - 0x90470 dans %eax
10	: push 		%eax 							> on met %eax dans la pile == %ebx - 0x90470
11 	: lea 		-0x90510(%ebx),%eax 			> chargement de l'adresse %ebx - 0x90510 dans %eax
12	: push 		%eax 							> on met %eax dans la pile == %ebx - 0x90510
12	: push 		%ecx 							> on met %ecx dans la pile == contenu de %esp
13 	: push 		%esi							> on met %esi dans la pile = contenu pop en 1 ?
14  : mov 		$0x8049db2,%eax	 				> on met l'adresse de <main> dans %eax
15 	: push 		%eax 							> on met main dans la pile
16 	: call 		804a0d0 <__libc_start_main>		> on appelle <__libc_start_main>
17 	: hlt										> on attend qu'un signal interrupt arrive
18 	: mov 		(%esp),%ebx						> on met le contenu de %esp dans %ebx
19	: ret 										> on retourne à la ligne 8
```


##Analyse de <check>

```
//stack setup
0	: push 		%ebp
1 	: mov 		%esp,%ebp
2 	: call 		8049e74 <__x86.get_pc_thunk.ax>> On met la position actuelle du code dans %eax
3 	: add 		$0x912f8,%eax 					> On ajoute 0x912f8 dans %eax

//On récupère le premier caractère
4 	: mov 		0x8(%ebp),%eax						
5 	: movzbl 	(%eax),%eax 					 
6 	: movsbl 	%al,%eax							
7 	: push 		%eax 							
8 	: call 		8049b8b <check_char_0> 			> On appelle <check_char_0>
9 	: add 		$0x4,%esp 						
10 	: test 		%eax,%eax 						> On lève le flag ZF si %eax == 0
11 	: je 		8049dab 						> On saute à la ligne 59 si %eax == 0

//On récupère le deuxième caractère
12 	: mov 		0x8(%ebp),%eax 					
13 	: add 		$0x1,%eax 						
14 	: movzbl 	(%eax),%eax
15 	: movsbl 	%al,%eax							
16 	: push 		%eax 					  
17 	: call 		8049bcf <check_char_1> 			> On appelle <check_char_1>

[...]

53 	: call 		8049ccb <check_char_5> 			> On appelle <check_char_5>
54 	: add 		$0x4,%esp 						
55 	: test 		%eax,%eax 						> On lève le flag ZF si %eax == 0
56 	: je 		8049dab 						> On saute à la ligne 59 si %eax == 0
57 	: mov 		$0x1,%eax 						> On met 1 dans %eax
58 	: mov 		8049db0 <check+0xb0> 			> On sort de la fonction
59 	: mov 		$0x0,%eax 						> On met 0 dans %eax
60	: leave 									> On remet la stack frame dans son état normal
61 	: ret 										> On quitte la fonction
```

On pourrait voir la fonction comme :
```
int check(char* chaine) {
	if (chaine[0] == ??) { 				//[Note à moi-même] Voir check_char_* pour comprendre
		if (chaine[1] == ??) {
			if (chaine[2] == ??) {
				if (chaine[3] == ??) {
					if (chaine[4] == ?) {
						if (chaine[5] == ?) {
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

##Analyse de <check_char_0>

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
9 	: movzbl 	-0x8(%ebp),%ebx		 			> %ebx <- %al sur 32 bits
10 	: sub 		$0x3f,%ebx 						> ebx -= 63
11 	: sub 		$0x74,%ebx 						> ebx -= 116
12 	: mov 		$0x4e,%edx 						> edx = 78
13 	: mov 		%ebx,%eax 						> eax = %al - 63 - 116 
14 	: imul 		%edx,%eax 						> eax = %eax * %edx = (%al - 63 - 116) * 78
15 	: mov 		%eax,%ebx 						> ebx = eax 
16 	: cmp 		$0xe0,%bl 						
17 	: je 		8049bc4 <check_char_0+0x39> 	> si %bl == \xe0, alors on saute ligne 20 
18 	: mov 		$0x0,%eax						> on met 0 dans %eax
19 	: jmp 		8049bc9 <check_char_0+0x3e>		> on saute ligne 21
20 	: mov 		$0x1,%eax 						> on met 1 dans %eax sinon
[//Sortie de fonction]
```

On pourrait voir la fonction comme (?) :
```
int check_char_0 (char a) {
	char ebx = (a - 63 - 116) * 78;
	int bl = ebx % 256; 		
	if (bl == 224) { 				//== -32
		return 1;
	}
	else {
		return 0;
	}
}
```
==> On veut bl == 224 
donc (a - 63 - 116) * 78 %256 == 224
donc (a - 63 - 116) * 78 == 224 + 256 * k
donc a = (224 + 256 * k)/78 + 63 + 116 

avec a = 'C' == 67, ça devrait marcher !


##Analyse de <check_char_1>

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
9 	: movzbl 	-0x8(%ebp),%ebx 				> %ebx <- %ebp - 8 == %al zero extended
10 	: shl    	$0x5,%ebx 						> %ebx << 5 //== %ebx = %ebx * 32
11 	: and    	$0xffffffef,%ebx				> On met à 0 le 5e bit de poids faible
12 	: test   	%bl,%bl 						> ZF à 1 si %bl == 0
13 	: je     	8049bfb <check_char_1+0x2c> 	> Si %bl == 0 alors on saute ligne 16
14 	: mov    	$0x0,%eax 						> Sinon met 0 dans %eax
15 	: jmp    	8049c00 <check_char_1+0x31> 	> Et on sort de la fonction
16 	: mov    	$0x1,%eax 						> On met 1 dans %eax 
[//Sortie de fonction]
```

On peut voir la fonction comme (?) :

```
int check_char_1 (char a) {
	char ebx = a;
	ebx = ebx << 5; 			//ou bien ebx = ebx * 32;
	ebx = ebx & 0xffffffef; 	//on met le 5e bit à 0
	int bl = ebx % 256;			//On récupère les 8 premiers bits
	if (bl == 0) {
		return 1;
	}
	else {
		return 0;
	}
}
```
on veut bl == 0,
donc ebx % 256== 0;
donc (a * 32) & 0xffffffef % 256 == 0

Donc a doit être tq. ses 3 derniers bits sont à 0 : l'opération AND n'agit théoriquement pas car celle-ci met à 0 un bit issu d'un shift
donc nécessairement à 0.

avec a = 'H' ou a = 'P' ou a = 'X', ça devrait marcher !

##Analyse de <check_char_2>

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
```
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
donc ebx = (a * 64) + a / 4 							//Aucun bit n'est commun donc c'est une addition

Avec a = '3', ça peut marcher ! 


###Analyse de <r> nécessaire

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

int r(int x, int y) {
	return (x << 8 - y&7) lor (x >> y&7);
}





##Analyse de <main>

```
//Setup de la stackframe
0 	: lea 		0x4(%esp),%ecx 					> chargement de l'adresse %ecx dans %esp + 4 (juste en dessous du point courant de la stack)
1 	: and 		$0xfffffff0,%esp				> on vire les 4 premiers bits de %esp
2 	: pushl		-0x4(%ecx) 						> empile l'entier %ecx - 4 == %esp au début du main
3 	: push 		%ebp 							> empile %ebp (bas de la frame)
4 	: mov 		%esp,%ebp 						> met le contenu de %esp dans %ebp : le bas de la pile devient %esp
5 	: push 		%ebx 							> empile %ebx
6 	: push 		%ecx 							> empile %ecx == contenu de %esp + 4 au début
7 	: sub 		$0x10,%esp 						> on retire 16 à %esp = on monte de 4 instructions 

8 	: call 		8049a10 <__x86.get_pc_thunk.bx> > on met la position actuelle du code dans %ebx
9 	: add 		$0x91237,%ebx					> on ajoute 0x91237 à %ebx
10	: mov 		%gs:0x14,%eax 					> ???
11 	: mov 		%eax,-0xc(%ebp)					> on met %eax 3 instructions au dessus de %ebp
12 	: xor 		%eax,%eax 						> on remet %eax à 0
13 	: movl 		$0x0,-0x18(%ebp) 				> on met 0 six instructions au dessus de %ebp
14 	: movl 		$0x0,-0x14(%ebp)				> on met 0 cinq instructions au dessus de %ebp
15 	: movl 		$0x0,-0x10(%ebp)				> on met 0 quatre instructions au dessus de %ebp
16	: sub 		$0xc,%esp						> on monte trois instructions au dessus de %esp 

[...]
```
