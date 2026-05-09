<script setup>
import { ref } from "vue";
import { jsPDF } from "jspdf";
import autoTable from "jspdf-autotable";

// Categorías
const categorias = ["Comida Rápida", "Bebidas", "Postres", "Platos Fuertes"];
const categoriaSeleccionada = ref("Comida Rápida");

// Estado
const mostrarAlerta = ref(false);
const pedido = ref([]);

// 30 PLATOS MANUALES
const platos = ref([
  { id:1, nombre:"Hamburguesa Clásica", categoria:"Comida Rápida", precio:15000, descripcion:"Carne a la parrilla", ingredientes:"Carne, pan, lechuga", imagen:"data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wCEAAkGBxIREhUTExMVFhUVGBgYFxcYFxYaGRgaFxoXGBkYGhgZHSggGSYlGxgYIjEhJSktLi4uFyAzODMsNygtLisBCgoKDg0OGxAQGy8lICY3LzUtMi8tLTU1Li03Ly0tMjEtMi8tKys1Mi0tLS0tLys1LTUtNy0tLS0tLSstLS0tLf/AABEIAP8AxgMBIgACEQEDEQH/xAAcAAEAAgMBAQEAAAAAAAAAAAAABQYDBAcCCAH/xAA/EAABAwIEBAUCAggEBgMAAAABAAIRAyEEBRIxBkFRYRMiMnGBkaEH8BQjQlKxwdHhFTNi8RZygpKisiRDU//EABoBAQADAQEBAAAAAAAAAAAAAAADBAUCAQb/xAAyEQACAgECBAQDBwUBAAAAAAAAAQIDEQQhEiIxUQUTQYFxofAUMmGR0eHxI0KCscEG/9oADAMBAAIRAxEAPwDhqIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIivHDv4Z4rEOpuqltGhUaHirLHmHU/EZDA4EzLQdoJXE5xgsyeDqMXJ4SKOi6vV/B39a7/AOY0UoGk6NVR3l8xLQQ0efYapg9lk4m/ClhNMYKoGkNa2p4pdpc6BNQOAJBLt2xF7QoXq6VjmJVprX/ackRdGxP4QYpryxmIwznNDi6XPaARENktuSDPICN1CZ7+HmYYTTqpeJrLgPBPinyt1SQ0SBEmY/ZKljdCWyaI3XOPVFURbOOwFag4NrUqlNxGoCoxzCRcSA4CRY37LWUhwEREAREQBERAEREAREQBERAEREAREQBTnC/CmKzCqKdFhiNTqjgQxrZIknncEAC5g9DERhMO6q9lNglz3BrR1c4gAfUr654fyxmAwlGlVqgmlTawvcY9Iiw5DoOnVQXWSiuX+DuEU3uQeSfh3gMOQ6lh26gZDny94MATqdttNo3MKyf4WF5/4qwQsKzf9l7p8SYRzh+uaDext06/m6z3XVLec8v4lxedH7sWl8DUrZedQGlukNub6i6emwELQxeDIb5A0O1CSQTYbgAEXKsn6fSds9pEDmOa03uBLj1EAe0X+6hs0kf7Sau+a6orr2kB0AEn96QN7m1zb+K16VTS4Fxtz02MdBdTdamXNsP+o7D43J7fdROJogDr3t+QqM6Zwxgv12RllMjWikaprOot8ZoLKTzocQw6gQXETs47dT1UJmn4cYHFnxWtdhp8sUi3wy/cu0v2gQSBClcQ4DaF6w+LB0tc5rRMy4w2wuT8D3upKNRZB4yeXaaua6HHsy/D7MKNRlMUfF8Rxax1EioCR10mWWvLo2PQxW8Vh3U3upvaWvY4tc07hzTBB9iF9K4etpJNMjp5XFsjpMT9lVuLeAKWKY51FrKNZ9TxH1HaiDIhzTps0SS6QLn7bFWtztNGXboXHeG5w5FKcQZDXwVU06zeul7bsqAftMdzG3cTcBRavJprKKLWNmERF6eBERAEREAREQBERAEREBMcHY1lDHYWrUjRTrU3OnYAOHm+N/hde4mqYqrWcxznuMnSGjfl5YF+a4lllA1K1NgbqLntGnrJFuy+jmZM+qBpeAYAjSZkWnlzCy/EYTk48Kya/hfCuKUml7HOa2W1XP0kGZAuLgEwTB/NlY38Js8Iv8dwHRzrk9gFOYfhGoKmqp52H1FrXTFzb5PJTAyOgaQBDabyL2LS09L3MWHQ3UUdK5RWXh9jSnqqq5LDz7HNauTFv+VingjkTMey1qWJzOgZpYkuAsZIIEnvBEwNls59qY9zXXAGrbeDyMTt16dCCa1XzGPS94+ZHYwQvIaa2PqmXpSoshlotLPxDxTBor0pbF3MNyfZ0d+alKXG2GxDoa8MsTD5Did4AiD03XNKmPcd3gnu0j/1n+CxVqlN4uyDzgiPpM/ZTOltYkvy/czZ11J5g8fH9jrLyH1A3UIhznf9I2+q8ZnimtbTcPDYNMNaJL3kGHVHECBcGxPIrlGEzOvQg03kNuADcbQYB2sfupnCcVNc0NrMDSOYvM9T/VV56RroReY09y/YTMriT8/1Vpy7GB9jGwgzv8cuX1XL6OLa67XW5KaweYupgGeYA/ifsD9lHDMNmdScZlu4l4YpYyj4Dmu8P1hzSAARPpExN5uIN/jiX4i8J/oNYeFTqCg5jIc6/nDQHyRsSbwepiwXbcDmvisLNbmHbU03BBUricpZiWFlRxqMfT0upugtdB9W1jfl2Vym1xfJ07FG+lS+917nyaiv3HX4c1cJUc7DsqVKLaZqPe4shkF0iZBMNAO0+6oK0oTUllGbKLi8MIiLo5CIiAIiIAiIgMuFw7qj202CXPcGtHUuMD7ro+UcA0WtDsQS4+5aNpkQQbd7+2y57lmOdQqtqs3afsQQR2sTfkrmePQ5oY2m4AyIIDjc2AJI2t9FR1ivaSq6epoaHyN3b19Mk3/g+DpPbUptLHMdLS3USDvIEw6Zgg8uatOB40LAPIHno1wDrXIDSAD9VRc2fUww1VWganEWA3aD0Npv9FH4XEV696VMNBMa3AAEnoTvtyVWMLeuX7mvw6fGMr2O/ZTxhTfSa51Gswm8OFOfcgPt9lgzfjWi1hhtUbSSGgdebrLgOJxmJptILgdJgFriCQbAC3m9x27KNp1MTXfoawlws655fvSeQtHZTyjZNPmWDNcKIS6PJ2Spx1h6hDBq81hAseZuOw3UZjMdg3nS6g1zrxNOTfoYVIyLLa1Kq59RgtTdFpANgfbyarqZxOYeCRVY0nRvAteZ+I+sKlOHDJRg8mhXDMHJrBq5vQwRFmNYRaxI5cxKp+OoabgyPupTMeIXuHpaGusARP5/uq34ztuR/Nlo6euaXMZ2ouinhGSm+eqyFxW3w7gGVarW1HaWE3Mx15rp+D4Cy+pq04mAGzaoxwkb3cI6WPe6mk0mQxcsbnI6GPew+Ukext9FZco4vNm1QIHO8cr9eQ2W5mPA37VOr6iS3UyJFiHSNgfba6h8w4PxFJupzHEjmwa2kWj0+YfIC4kqrNmerzobnQsmzRpaXsdcd95tP1hXTIc62B5WXzxlmPqUXAsd8Tur/wAK8VNqOh5hxOx6n+6p2UyqfFHoWoWKawzqnEGVNxlF9F0/r2Oa6o0kaNMlhgczMHkQIK+auIsndg676LtRDSQHlhYHgGC4A8pBE9l9G5PmW4nYAQev5IHwei5h+OFDz4aqWkOe1wNzAADDpjYHU55kbz2U2mt58dytqauTi7HLkRFoGeEREAREQBERAFK5PkWLrlrqGGqVBIghhLJB5u9Me5V4/B3hWjX8TGYgNdTpO0sa+NGoAOc5wNjALYm0nsuv084wjjpbiaJIgQHttta3uPqsDxDxqVNjqphxNdX6L9S1TpnNZZzTO8tqVKbGYtoY5o1GAHXPmLQRynULdVs4OvTOG0U6etz2gBktBdq5gTad5EwJO4hdEzDJm1mRUZqaff7EbKnZvwu+iRUw5c8CJpiGPHqB0v1AOsRvHp5yqmi8YUlw27P8dl7ZNJxhPp1IQcMurYum03c1uouaCGuLQ0iAANMPtfcE9LZcVRZhDcgblwLZiB0+p+i/cvzd2GcW1Gnxi2W6tvU0AEk22n47qtcR8QOe4NI1OduXG3mi5G+4+y0rF5skorK+R3W1VFuT/UmcRxJRA/ZP3JneYtf3t/Gm5lxCwvLmsntM+08vsozHms4S6AOgtveI/M/CxYfLXPbIkm9gLNgSS49ALqxVpIQ3ZXu1k3tBYPNXFuqVBUc0EA+kC0cxAVppYfB1QABu0mBeNIki51D2P12Wp/gzqTREtMS559IiJ39V3CwX7hse8OexzA8MkOdDoG4AkA7xA2XVuWuT0/E4qfA+ffJqV8hpuk0qmr91sgn+E/C1xgqlJ0G3IFtpPfmO3WFJChRdUDXtdRJuDIIG3fa/RZ65ex1qzKnK+5EzF5tbaVz5zXK/n+vQkVNbeYbGDLsxrNgtrE32cZHff+ys7uJ20WS6D6B5QZv0nsqhWw4F4NMkzMEgxy3hYmY/U8tqxDtgD5Y/MX7JwqW6JXJKPDJb+jL5h6mAzAfrGNc6DLm+WqN78ifYyLKq8TcOnDEVKbtTJN9iLSJ6+/ORsseXuZRfrLCN9JtHYgRc+ylc2qnEllNpIa8NN+TtiL7gwD8p5mJY9Dz7Py5N/gnicktp1TcEQ49LiD15fRbn41vBw+GNp8R0e2m/8vqqjSy+pTqhpaZ3sDHMb+6/fxKzbxalCj/+FIB3/O8NLh8NDB7grmqCdyx8SLWYhX16lNREWkZAREQBERAFYOHeFKuMYajXBrWvDTYknYuI9gRHU2ULhAwvYKhIZqGsjcNm5HeF2LIm020D+j0mihUJLXB8SfSSC4zIgW7Khr9RZVD+mt/kcTlwo9ChToYKjhmEimwuLy6xc8uLvNp+3UBSGVspubNmNEuaZaBpMOF4taAfe+5UBiK1WXtBDmGA6biQRckbeYTK94x4oMLKrdVGA0EDVYCzXAneRY84+FQ0bjFtPq237vqaWkssurfaODZfmeLY6Kdao2k4fqn7nyl2p5a4Hy9PZSmS8YOb5cY3VG9Vpa0C8S7UQN99JNzsojB1KQY17yC5jACJLi0AQ4SCYuCJmCAd7L3hMgZVotOIu+sGMc1rQGsAMhrdMHdgnfafexf4bprVjhXsTRuktmWyvl2X5m0PZVa5w2LXQ4fT63VT4n/DOsYdRqAneHCD1EECPstHMCcPqMhzW+k/ulpcC4RABaR9AdrFWDK+M3YbCuqVyatLdjiZc3UB5RNz1usx6LU6bm08spdE9/r2wS8fFs/r3KBxDlGIpEOqUHBogOiXNItcETERz6LWyHMHURUrAAkAhggRLi3U6LcmwJ/eXWMp41wldj/GmkWFo1OHkIfYQ7n9F7zPhrB4keZgcDcOaSPYy33P1ReLWUYhqa3H8V9f9OvKU3mJyJ9d7sOQwavELi5zgZEEGNRsDIbYC9t4C0eGa36xzHP0tc2XOguIDbiB1F/7ro2c/h3FLRh6hEElrX3HmEEAjqqYzhrE4NlRz6d4EOaC4ASJiAYMTBI/vqabxHT3RfDJZ7PqVraJJp4JXNHtaNZb4dRhDPDkEtt5WD94mYjoy+6jqdEPJBjxLAMcYLjqAcIEgGNXUWCruZZgXuLmktEgiJkEbGd5C3cHWp0xTcHEP3OnlBBu4eYk/burfB6nPHvhFwyTM6LHPoYim3wn2E3AiA4GbiDsehBU3X/DfCVxrpOcBFoMwP6KluxVLS0g3eGkTN9BDdO8SBbfe/QKf4O4oOHxDaLnHRUAA5aXxe3LVBt1HdZWt0lkM3USa7pf7J6rlLln8ymcWcP4nA1tL5czem8A6S07gDkRzHz3W9kmdMeNJhr4Ed3NEBwnqDcdQN11vi/OKdSl4QpU6lSJv6RaxK4fnWBd4hqtpaaboMC4G1+09O6taS2V9a41/l0z7fS7EbzU8p/FHUDRFSmA6Wv0y13v6bkbxBvzHNcczbCVKVVzakl0zqP7QNw6ec7q6cO5m/wtLajjouab7gt5uY4XBEyWmYBWPjZzX0BrABYQaT5klr/VTnnBkiYI0nqpabFCzhO9TQ7KvMXoUNERaBkBERAEREB+tIkTcdF1ThvNKVUaKFJ1OjTLg0uuCXCXAXJBl08/cbLlS6d+GNPDjB16lYlv63TJMNIDQZj/AE3k/wCsdlR19SnVnscTWUWjKcEys7S+nIkSWixi4mCFu51lYpuLDdjhIvaOnwbKHyTNWOqFtBjQ8AuhzpceQLWzzMBZznVQSzFNIaHENOomNiHtdzB/ksPy3GPTdGn4dq/JnlrlezIHOOHq7NT6WqpRd6tI87dI2MTbeZmZMpieIHuDNwGSf9WsA6RHUCT/ALK95HmJpAlwLqViKjbgW5gXAgC+23VRnGmBwmLDagaNXJ7CWu33PI/IPaFrVWpwTbL04Rsn/TW3c55xFmznUWDzkAadWktDoDSJBMjfVHU9lI8Jik/Bt/SGmp5xoBgBmgkDuZ1H/tnktHNqVSlSdTgVmEQ15nW0HeY3/sFoZPxCadI03OeXMDvDEt07ftTuQJgeyswkpRzEqW1yhPEzPxI6o46n7P1HTEEDY+U+xg9pgK98AZ43DUm0q3+Q4xTcTemCRp1TsCTIm4BHI25Rjse+o6XGOcmTJv8AxJP1Ui/OHOpgVHNAY1g0j1PgAMYTyi5J+N4UWo0kb6vLkjxWJSyfQeIOlaGIxbNnELlNP8R65woptpzUpN0mo51i3ZpI3JiBvylV6pj6+JJFWq+TNpIaIvdojbuvm6f/AD9uX5jwi19qgo7bs6BxFlmX1iZNMP6yAVQc04eNM/qnB7eQlY6FFgaNYZMkERcEGI/n7ELYxGHaaRq0n6HMIkNJFtpI5re02lso5VNv4leyyM1xNEHiS8FrS0t08j16rJXxessuZaRLgTNo0xexF/zCnKeNqaCHuYfSJcwH1mPVbp/dY25W0vHi0tJJjyEA9jHKb/RXFOSXMvyIOHOyZhw+cVafm8QlxcCS4mXBrrOPxa52HZSVPOw5pGkNDmkx5iAS52q02nfnEd15fwqKjyKL3H0+V4AdcTuPVaFsf8F4yHOFOQxpd5TLjHPkoXqaovDeH+OxKq59cEPWrCnrLYAHmA7h0RbnDnD4C2qGdUKlJ1OrdhBMGNTXctPfuPlVjE4nUIExz73WsrD06lhvqRLWShmMej9GERFYKQREQBERAFLYDP6tKk6kAxzXBzRrBOnXvpvHPmColFzKMZbNDBv5DmJw2IpVxfw3tcR1E+YfIkLr2d4SlWDH0ZextKSdxpA1MIPs6/suJK5cCcaHBzRrS7DuBEblmreOx5j56zT1unlYuKHVE9Uk15cnhZydSybK6jKFKuyvppn/AOrSAzzydMnlffqojO8fhWVPDBd4jw4zBDZDSZ07AWP9lZsmxbG0NWGc19NzSQSZE8h+doXNuI2tfX8Rr3te2zSDsb7GZA3WQpKc1Hdd/j9dTu9T09i8p9e3Yy1i9rQXtDSQCRIJE7agCS2RcTCruOwrHkktC2uH8ge7ECpUFTSQ506naqhO0uBnczJ3085UrhM2oOrHxaMPZqAaRLW+GHHUDHmsOfMDqCrW1cn5eXjt/Jeh4wpR4bY5KViMCNg7bZadSk4d1vYvEO1Fz93EuJFx5r+6Yak6rIp+bsD8DeOZWlGUkssislprN47Efhqpa6SJHMXg+62W457J0mJnpzt8be68vb2/PwsdOkXkMaCS4wB1KkymVmuFYTPVKpLi50X5x/JbFLHljHANadQF/Yg7cxFo/ovWJygtY57HioGEB4AILZ2In1Cef2UZqISLUuhFGaa2LPlObMczwntGkkHrtP8AI/YL1isS2nVY+nUBDjcEExoDQJG42NwTbuL17B0HuJ8NpcQJI7bfzj5XkayRY22CYR2p5+JeaubNo1KdeXgkjTpeC0elpJESdOmQ2d57qfq8V4ljmvNYPaILwWadQO/hn1H6kbdZXOqGNqWYGa9UiC0uJdNi0QSCCTte6ncDltRzx+kOdSY1rmwZ16XcmtvBnrCqX1VyXOkTfaFUstlf41wzaeOrtbZuvUB01gPj/wAlCKwcT5E6gBWNRz21HEAv9f8ApLr3kT/2qvq1Rjy4pPO3UpOam3JeoREUx4EREAREQBERAEREBL5DxFWwhOgyx3rpunS4bH2McwrU3PsJiqYZp8B4MwTIJ6h53+brnyKtdpYWPPR90T1Xyrfc6xhcXiKWlzXl0CJJJgdr233HVRmc02im409Rc4hkRe8O3HeFSsBnNajZriW/um4+OimMPxK12kVWmAZttO3K+ypPS2wlnqizKyi6Di+VskHZcPKHt0vABLY+p/ifhbAwlNlN9Snpljqb7iGvc19gBMmxcfYSvGEr4eDUa4F9yOuq8c+XQ27LTbjHQG2dYb7iBG/wvFxN+pE9BDCaka+EEPJDBTphwdUuXuqTqOiY2MG1hfmvzA5RUGGqVaTAXtLtTjOrwyADp5CPNPODvyW1SzBmoy2BGkjkfn+a2cZxC59M0qZbSpusWNBBI6FxJJ79VLxzzsitPS2ce3QjuHns8F9PXprVXxcSIA8gje5c6SOgUQ7BOLiHzv8AQ81J4PQyqx5jymfmLH4N/hSDxQ9T3yXGzWiY7nb6C/ZdOxxk2vUisqsi+VHjLcG+AyjDtUa7EHeQByjqSsdKjTqOqFmovcXOBtEyTpAm4N4Ntx8zDMzosaabGh3iNcKjvMAGR6ZaQSTtY845rQy7DHSX0CQRuwwbc4cRIj+qh42k29m/rcrKVlLytm/mbeU+FSqziASRIF4lwkRsbDrfktvMWGz6NNkkiYaJgmx2681F4zMG+PToO8/iPDnt5M1dCOcEmbQIW3Vz3Bsqmk41WNbbVu0gi4cBeL9915wTk1t/B1bO2775o8YXw7TUs8OAbpIIMCCHRYQJj2KpCs3E+MoOBbSqeIC4PFiI33JA6myrKvaWtwhhktKahhhERWSUIiIAiIgCIiAIiIAiIgCIiA/WuIuFnGNqfvLXReNJ9T1Sa6GwMW5ev0w9FqovOBHXmS7m43G9QsrMc3oo5Fy64s6V0iXZm4b+zPXlP9FaOHuL8NQdpcww4f5hHo7aRMzzI/qqAi4eng3khsSm8yJnOs1acY6vQEAEaZG8CJg8j0UTWque4ucZJMk9yvCKZRSCWAiIvT0IiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiIAiIgCIiAIiID9X4iIAiIgCIiAIiIAiIgCIiAIiID/9k=" },
  { id:2, nombre:"Pizza Pepperoni", categoria:"Comida Rápida", precio:20000, descripcion:"Pizza italiana", ingredientes:"Queso, pepperoni", imagen:"data:image/jpeg;base64,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" },
  { id:3, nombre:"Perro Caliente", categoria:"Comida Rápida", precio:12000, descripcion:"Hot dog clásico", ingredientes:"Salchicha, pan", imagen:"https://source.unsplash.com/400x300/?hotdog" },
  { id:4, nombre:"Papas Fritas", categoria:"Comida Rápida", precio:8000, descripcion:"Papas crujientes", ingredientes:"Papas, sal", imagen:"https://source.unsplash.com/400x300/?fries" },
  { id:5, nombre:"Sandwich", categoria:"Comida Rápida", precio:10000, descripcion:"Sandwich mixto", ingredientes:"Jamón, queso", imagen:"https://source.unsplash.com/400x300/?sandwich" },
  { id:6, nombre:"Tacos", categoria:"Comida Rápida", precio:14000, descripcion:"Tacos mexicanos", ingredientes:"Carne, tortilla", imagen:"https://source.unsplash.com/400x300/?tacos" },
  { id:7, nombre:"Arepa con Queso", categoria:"Comida Rápida", precio:9000, descripcion:"Arepa tradicional", ingredientes:"Maíz, queso", imagen:"https://source.unsplash.com/400x300/?arepa" },
  { id:8, nombre:"Nuggets", categoria:"Comida Rápida", precio:13000, descripcion:"Pollo crujiente", ingredientes:"Pollo empanizado", imagen:"https://source.unsplash.com/400x300/?chicken-nuggets" },

  { id:9, nombre:"Jugo Natural", categoria:"Bebidas", precio:8000, descripcion:"Jugo de frutas", ingredientes:"Frutas frescas", imagen:"https://source.unsplash.com/400x300/?juice" },
  { id:10, nombre:"Malteada", categoria:"Bebidas", precio:12000, descripcion:"Malteada cremosa", ingredientes:"Leche, helado", imagen:"https://source.unsplash.com/400x300/?milkshake" },
  { id:11, nombre:"Café", categoria:"Bebidas", precio:5000, descripcion:"Café caliente", ingredientes:"Café", imagen:"https://source.unsplash.com/400x300/?coffee" },
  { id:12, nombre:"Chocolate Caliente", categoria:"Bebidas", precio:7000, descripcion:"Chocolate caliente", ingredientes:"Chocolate, leche", imagen:"https://source.unsplash.com/400x300/?hot-chocolate" },
  { id:13, nombre:"Gaseosa", categoria:"Bebidas", precio:4000, descripcion:"Refresco", ingredientes:"Gas", imagen:"https://source.unsplash.com/400x300/?soda" },
  { id:14, nombre:"Agua", categoria:"Bebidas", precio:3000, descripcion:"Agua natural", ingredientes:"Agua", imagen:"https://source.unsplash.com/400x300/?water" },
  { id:15, nombre:"Limonada", categoria:"Bebidas", precio:6000, descripcion:"Limonada fresca", ingredientes:"Limón", imagen:"https://source.unsplash.com/400x300/?lemonade" },

  { id:16, nombre:"Pastel de Chocolate", categoria:"Postres", precio:10000, descripcion:"Pastel dulce", ingredientes:"Chocolate", imagen:"https://source.unsplash.com/400x300/?cake" },
  { id:17, nombre:"Helado", categoria:"Postres", precio:7000, descripcion:"Helado frío", ingredientes:"Leche", imagen:"https://source.unsplash.com/400x300/?ice-cream" },
  { id:18, nombre:"Brownie", categoria:"Postres", precio:8000, descripcion:"Brownie", ingredientes:"Chocolate", imagen:"https://source.unsplash.com/400x300/?brownie" },
  { id:19, nombre:"Cheesecake", categoria:"Postres", precio:11000, descripcion:"Torta de queso", ingredientes:"Queso crema", imagen:"https://source.unsplash.com/400x300/?cheesecake" },
  { id:20, nombre:"Donas", categoria:"Postres", precio:6000, descripcion:"Donas dulces", ingredientes:"Harina", imagen:"https://source.unsplash.com/400x300/?donuts" },
  { id:21, nombre:"Flan", categoria:"Postres", precio:7000, descripcion:"Postre cremoso", ingredientes:"Leche", imagen:"https://source.unsplash.com/400x300/?flan" },
  { id:22, nombre:"Gelatina", categoria:"Postres", precio:5000, descripcion:"Postre ligero", ingredientes:"Gelatina", imagen:"https://source.unsplash.com/400x300/?jelly" },

  { id:23, nombre:"Arroz con Pollo", categoria:"Platos Fuertes", precio:18000, descripcion:"Plato tradicional", ingredientes:"Arroz, pollo", imagen:"https://source.unsplash.com/400x300/?chicken-rice" },
  { id:24, nombre:"Carne Asada", categoria:"Platos Fuertes", precio:25000, descripcion:"Carne a la parrilla", imagen:"https://source.unsplash.com/400x300/?steak" },
  { id:25, nombre:"Pollo a la Plancha", categoria:"Platos Fuertes", precio:20000, descripcion:"Pollo saludable", imagen:"https://source.unsplash.com/400x300/?grilled-chicken" },
  { id:26, nombre:"Pescado Frito", categoria:"Platos Fuertes", precio:22000, descripcion:"Pescado crujiente", imagen:"https://source.unsplash.com/400x300/?fried-fish" },
  { id:27, nombre:"Spaghetti", categoria:"Platos Fuertes", precio:17000, descripcion:"Pasta italiana", imagen:"https://source.unsplash.com/400x300/?spaghetti" },
  { id:28, nombre:"Lasaña", categoria:"Platos Fuertes", precio:19000, descripcion:"Lasaña casera", imagen:"https://source.unsplash.com/400x300/?lasagna" },
  { id:29, nombre:"Sopa de Pollo", categoria:"Platos Fuertes", precio:15000, descripcion:"Sopa caliente", imagen:"https://source.unsplash.com/400x300/?soup" },
  { id:30, nombre:"Bandeja Paisa", categoria:"Platos Fuertes", precio:30000, descripcion:"Plato típico", imagen:"https://source.unsplash.com/400x300/?colombian-food" }
]);

// FILTRO CORRECTO
function platosFiltrados() {
  return platos.value.filter(p => p.categoria === categoriaSeleccionada.value);
}

// FUNCIONES
function agregar(plato) {
  let existe = pedido.value.find(p => p.id === plato.id);
  if (existe) existe.cantidad++;
  else pedido.value.push({ ...plato, cantidad: 1 });
}

function eliminar(i) {
  pedido.value.splice(i, 1);
}

function total() {
  return pedido.value.reduce((sum, p) => sum + p.precio * (parseInt(p.cantidad)||0), 0);
}

// PDF
function generarFactura() {
  if (pedido.value.length === 0) {
    mostrarAlerta.value = true;
    setTimeout(() => mostrarAlerta.value = false, 3000);
    return;
  }

  const doc = new jsPDF();

  const rows = pedido.value.map(p => [
    p.nombre,
    p.cantidad,
    "$" + p.precio,
    "$" + (p.precio * p.cantidad)
  ]);

  autoTable(doc, {
    head: [["Producto", "Cant.", "Precio", "Total"]],
    body: rows
  });

  doc.text("TOTAL: $" + total(), 140, doc.lastAutoTable.finalY + 10);
  doc.save("factura.pdf");
}
</script>

<template>
  <div>
    <h1>🍽 Sistema de Mesero</h1>

    <select v-model="categoriaSeleccionada">
      <option v-for="cat in categorias" :key="cat">{{ cat }}</option>
    </select>

    <!-- 🔥 ARREGLO AQUÍ -->
    <div class="platos">
      <div v-for="plato in platosFiltrados()" :key="plato.id" class="card">
        <img :src="plato.imagen" />
        <h3>{{ plato.nombre }}</h3>
        <p>{{ plato.descripcion }}</p>
        <small>{{ plato.ingredientes }}</small>
        <p class="precio">${{ plato.precio }}</p>
        <button @click="agregar(plato)">Agregar</button>
      </div>
    </div>

    <div class="pedido">
      <h2>🧾 Pedido</h2>

      <p v-if="pedido.length === 0">No hay productos</p>

      <div v-if="pedido.length > 0">
        <div v-for="(p,i) in pedido" :key="i" class="item">
          <span>{{ p.nombre }}</span>
          <input type="number" v-model="p.cantidad" min="1" />
          <span>${{ p.precio * p.cantidad }}</span>
          <button @click="eliminar(i)">❌</button>
        </div>
      </div>

      <h3>Total: ${{ total() }}</h3>

      <button @click="generarFactura">📄 Descargar Factura</button>
    </div>

    <div v-if="mostrarAlerta" class="alerta">
      ⚠️ No hay productos en el pedido
    </div>
  </div>
</template>

<style>
body {
  background: radial-gradient(circle at top, #151821, #0f1115);
  color: white;
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 20px;
}

/* TÍTULO */
h1 {
  text-align: center;
  font-size: 32px;
  margin-bottom: 20px;
  background: linear-gradient(90deg, #ff3b3b, #ff7a18);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* SELECT CATEGORÍA */
select {
  display: block;
  margin: 0 auto 25px auto;
  padding: 10px 15px;
  border-radius: 10px;
  border: none;
  background: #1c1f26;
  color: white;
  font-size: 14px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.4);
  outline: none;
}

/* GRID PRODUCTOS */
.platos {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(230px, 1fr));
  gap: 22px;
}

/* CARD PRODUCTO */
.card {
  background: linear-gradient(145deg, #1c1f26, #14171d);
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 8px 20px rgba(0,0,0,0.5);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 25px rgba(0,0,0,0.7);
}

.card img {
  width: 100%;
  height: 150px;
  object-fit: cover;
}

.card h3 {
  margin: 10px;
  font-size: 18px;
}

.card p {
  margin: 5px 10px;
  font-size: 13px;
  color: #ccc;
}

.card small {
  margin: 0 10px;
  display: block;
  color: #888;
}

.precio {
  font-weight: bold;
  color: #ff3b3b;
  margin: 10px;
}

/* BOTÓN */
.card button {
  width: 90%;
  margin: 10px;
  background: linear-gradient(90deg, #ff3b3b, #ff7a18);
  border: none;
  color: white;
  padding: 10px;
  border-radius: 10px;
  cursor: pointer;
  font-weight: bold;
  transition: 0.2s;
}

.card button:hover {
  transform: scale(1.05);
}

/* PEDIDO */
.pedido {
  margin-top: 40px;
  background: linear-gradient(145deg, #1c1f26, #14171d);
  padding: 25px;
  border-radius: 18px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.6);
}

.pedido h2 {
  margin-bottom: 15px;
}

/* ITEM PEDIDO */
.item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #0f1115;
  padding: 10px;
  border-radius: 10px;
  margin-bottom: 10px;
}

.item input {
  width: 50px;
  text-align: center;
  border-radius: 5px;
  border: none;
}

/* BOTÓN ELIMINAR */
.item button {
  background: transparent;
  border: none;
  color: red;
  font-size: 18px;
  cursor: pointer;
}

/* TOTAL */
.pedido h3 {
  margin-top: 15px;
  color: #ff7a18;
}

/* BOTÓN FACTURA */
.pedido button {
  margin-top: 10px;
  width: 100%;
  background: linear-gradient(90deg, #ff3b3b, #ff7a18);
  border: none;
  padding: 12px;
  border-radius: 12px;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

/* ALERTA */
.alerta {
  position: fixed;
  top: 20px;
  right: 20px;
  background: #ff3b3b;
  padding: 15px 20px;
  border-radius: 10px;
  box-shadow: 0 10px 20px rgba(0,0,0,0.4);
}
</style>