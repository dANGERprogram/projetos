import tkinter as tk
from tkinter import messagebox
from datetime import datetime
import os

MARGEM = 66

def gerar_gcode(altura, largura):
    x = altura - MARGEM
    y = largura - MARGEM

    gcode = f"""G21
G90
G0 X66.00 Y66.00
G1 Z-6.00 F300
G1 X{y:.2f} Y66.00
G1 X{y:.2f} Y{x:.2f}
G1 X66.00 Y{x:.2f}
G1 X66.00 Y66.00
G0 Z5.00
"""
    return gcode, x, y

def salvar_nc(gcode, altura, largura):
    pasta = "arquivos_nc"
    os.makedirs(pasta, exist_ok=True)

    data = datetime.now().strftime("%H%M%S")

    nome = f"porta_{altura}x{largura}_{data}.nc"

    caminho = os.path.join(pasta, nome)

    with open(caminho,"w") as f:
        f.write(gcode)

    return caminho

def desenhar_porta(canvas, altura, largura):

    canvas.delete("all")

    escala = min(400/largura,500/altura)

    w = largura * escala
    h = altura * escala

    x0 = (500-w)/2
    y0 = (600-h)/2
    x1 = x0 + w
    y1 = y0 + h

    # MDF externo
    canvas.create_rectangle(
        x0,y0,x1,y1,
        fill="#d7b38a",
        outline="#8a5a2b",
        width=3
    )

    margem = MARGEM * escala

    xi0 = x0 + margem
    yi0 = y0 + margem
    xi1 = x1 - margem
    yi1 = y1 - margem

    # sombra
    canvas.create_rectangle(
        xi0,yi0,xi1,yi1,
        fill="#cfa57a",
        outline="#704321",
        width=2
    )

    # relevo interno
    canvas.create_rectangle(
        xi0+6,yi0+6,xi1-6,yi1-6,
        outline="#efcfb2",
        width=2
    )

    # pontos 66
    r=5
    for p in [(xi0,yi0),(xi1,yi0),(xi0,yi1),(xi1,yi1)]:
        canvas.create_oval(
            p[0]-r,p[1]-r,
            p[0]+r,p[1]+r,
            fill="blue"
        )

def gerar():

    try:

        altura = float(entry_altura.get())
        largura = float(entry_largura.get())

        gcode,x,y = gerar_gcode(altura,largura)

        txt.delete("1.0",tk.END)
        txt.insert(tk.END,gcode)

        caminho = salvar_nc(gcode,altura,largura)

        lbl_result.config(
            text=f"X = {x:.2f}   Y = {y:.2f}\nArquivo salvo:\n{caminho}"
        )

        desenhar_porta(canvas,altura,largura)

    except:
        messagebox.showerror("Erro","Digite valores válidos")

janela = tk.Tk()
janela.title("Gerador Porta Almofada - Incantare")
janela.geometry("900x650")

top = tk.Frame(janela)
top.pack()

tk.Label(
    top,
    text="GERADOR DE PORTA ALMOFADA",
    font=("Arial",18,"bold")
).pack(pady=10)

frame = tk.Frame(janela)
frame.pack()

tk.Label(frame,text="Altura").grid(row=0,column=0)
entry_altura = tk.Entry(frame)
entry_altura.grid(row=0,column=1)

tk.Label(frame,text="Largura").grid(row=1,column=0)
entry_largura = tk.Entry(frame)
entry_largura.grid(row=1,column=1)

tk.Button(
    frame,
    text="GERAR",
    command=gerar,
    bg="green",
    fg="white",
    width=20
).grid(row=2,column=0,columnspan=2,pady=10)

lbl_result = tk.Label(janela,text="")
lbl_result.pack()

canvas = tk.Canvas(janela,width=500,height=600,bg="white")
canvas.pack(side="left",padx=20,pady=20)

txt = tk.Text(janela,width=40,height=20)
txt.pack(side="right",padx=20)

janela.mainloop()
