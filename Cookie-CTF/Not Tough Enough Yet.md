On a un binaire classique, on le met dans Ghidra pour voir un peu son contenu.

<img width="1570" height="862" alt="image" src="https://github.com/user-attachments/assets/2a36785d-0ca5-4f89-ad03-757051061ed8" />

Ici on voit le flag, qui s'est très clairement fait xoré. Plus qu'a trouvé la clé XOR et on devrait pouvoir récupérer ça en clair.

On cherche un peu, même si pas vraiment longtemps vu que il y a une grosse fonction "xorinput" sous nos yeux qui contient ceci : 

<img width="1564" height="856" alt="image" src="https://github.com/user-attachments/assets/14750f90-ede0-498a-818d-fffdd365516a" />

Ce qui ressemble quand même vachement à une clé xor.

Donc on fait un petit code python avec tout ça : 

```python
a = [0x02, 0x3d, 0x2a, 0x1c, 0x2c, 0x0d, 0x1a, 0x2e, 0x26, 0x0d, 0x26, 0x1b, 0x7e, 0x01, 0x2d, 0x5f, 0x21, 0x53, 0x45, 0x23, 0x41, 0x32, 0x63, 0x1c, 0x05, 0x0e, 0x11, 0x0e, 0x31, 0x42, 0x06, 0x28, 0x1a, 0x0b, 0x1d, 0x55, 0x1e, 0x5c]
b = [0x41, 0x72, 0x65, 0x57, 0x65, 0x48, 0x61, 0x63, 0x6b, 0x65, 0x72, 0x73, 0x4f, 0x72, 0x6e, 0x6f, 0x74, 0x3f, 0x21]
flag = [chr(a[i] ^ b[i % 19]) for i in range(38)]
print(''.join(flag))
```

et on obtient ce truc sympa : 

<img width="899" height="33" alt="image" src="https://github.com/user-attachments/assets/cc80f07e-98ea-4bdd-8077-7ebb0cd10299" />
