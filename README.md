# sheet_sub
Sub CopiarParaM()
    On Error GoTo erroHandler
    Dim btn As Button
    Set btn = ActiveSheet.Buttons(Application.Caller)
    
    Dim btnCell As Range
    Set btnCell = btn.TopLeftCell
    
    Dim colunaOrigem As Range
    Set colunaOrigem = btnCell.EntireColumn.Range("5:17")
    
    colunaOrigem.Copy Destination:=Sheets("Planilha1").Range("M5")
    Exit Sub

erroHandler:
    MsgBox "Erro ao tentar copiar a coluna. Verifique se está utilizando botões de formulário."
End Sub


Sub CopiarParaN()
    On Error GoTo erroHandler
    Dim btn As Button
    Set btn = ActiveSheet.Buttons(Application.Caller)
    
    Dim btnCell As Range
    Set btnCell = btn.TopLeftCell
    
    Dim colunaOrigem As Range
    Set colunaOrigem = btnCell.EntireColumn.Range("5:17")
    
    colunaOrigem.Copy Destination:=Sheets("Planilha1").Range("N5")
    Exit Sub

erroHandler:
    MsgBox "Erro ao tentar copiar a coluna. Verifique se está utilizando botões de formulário."
End Sub
