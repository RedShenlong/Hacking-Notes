# Enumeración de HotFixes en Windows

La enumeración de **HotFixes** (actualizaciones del sistema) es una fase importante durante la auditoría de un sistema Windows. Permite identificar qué parches de seguridad están instalados y detectar posibles vulnerabilidades conocidas que puedan ser aprovechadas para realizar una escalada de privilegios local.

---

# Enumeración mediante PowerShell

## Listar todos los HotFixes instalados

Muestra una lista con el identificador del parche (KB), descripción, usuario que realizó la instalación y fecha.

```powershell
Get-HotFix
```

## Contar el número total de HotFixes

Permite conocer rápidamente el nivel de actualización del sistema.

```powershell
(Get-HotFix).Count
```

---

# Enumeración desde CMD

Si no es posible utilizar PowerShell, puede emplearse **WMIC** para consultar las actualizaciones instaladas.

## Listar HotFixes

```cmd
wmic qfe list brief /format:table
```

> **Nota:** `WMIC` ha quedado obsoleto en las versiones modernas de Windows. Siempre que sea posible, se recomienda utilizar `Get-HotFix` desde PowerShell.
