# Old Data

<img width="647" height="245" alt="image" src="https://github.com/user-attachments/assets/0c671f48-cb05-48c3-b06d-0784d3ee46ed" />

```
crea@DESKTOP-T3UPM05:~/CTFs/Cookie$ strings old
-%/7
4EMPLE
!NCIENTS
'=<?:$
-%/7
 47.-
-%/7
 3//0&
-%/7
#*0?8!
-%/7
!3132)
-%/7
+#//+)%[TH
&4198"
%1.:8"
?GRE
#3715!
_bdh
4EMPLE
!NCIENTS
#LOUD
3ECTOR
3LUMS
#LOUD
"ARRET
4IFA
!ERITH
j2ED
+9UFFIE
#LOUD
2UHDFA.
3EPHIROTH
?#ID
dddd
3ECTOR
3LUMS
IDGAR
```

Strings et xxd montrent des trucs en rapport avec ff7 (les noms des persos et le lieu) + on dirait des stats. Donc je fouille un peu google pour voir comment on peut lire des vielles saves ff7 ou si c'est quelque chose en rapport avec un mod par hasard.
Après quelques recherches, je privilégie la pite du fichier de save. Donc je download Black Chokobo, un truc qui nous permet de load et visualiser des fichiers de save ff, j’essaie de load le file dedans mais ça marche pas. Je regarde comment s’appelle les fichiers de save ff7 habituellement et je le rename save00.ff7 et la ça load dedans.

<img width="919" height="710" alt="image" src="https://github.com/user-attachments/assets/3a308fc2-9407-4a98-9393-5d2edfe0d547" />

Ensuite j’explore la save 1, et en regardant le nom des différents persos : 

<img width="915" height="683" alt="image" src="https://github.com/user-attachments/assets/92dcad05-1704-4b13-929b-ff4b5a5e90d4" />

Je vois que petit a petit ça forme un flag

```
COOKIE{th4t_w4s_h3ll_0f_4_gre4T_g4M3}
```
