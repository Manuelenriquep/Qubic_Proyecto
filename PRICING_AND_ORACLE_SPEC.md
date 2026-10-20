# Especificación de Servicio y Modelo de Precios: Oráculo Determinista B2B para Qubic

**Versión:** 2.0  
**Arquitectura:** Caja Negra Determinista (Kernel Nivel 0 + Inferencia Activa)  
**Protocolos Soportados:** gRPC (Ultra-baja latencia en C-ABI) / REST JSON (HTTP/2)  

---

## 1. Esquema de Micro-tasas por Llamada (*Pay-per-Proof*)

El oráculo opera bajo un modelo de facturación instantánea por prueba verificada (*pay-per-proof*), liquidada en el token nativo de Qubic (\$QUBIC) o moneda estable equivalente.

| Vertical de Aplicación | Caso de Uso Crítico | Micro-tasa Estimada (\$QUBIC) | SLA Latencia de Verificación | Política de Fallo |
| :--- | :--- | :---: | :---: | :--- |
| **Finanzas Descentralizadas (DeFi)** | Liquidación instantánea, arbitraje libre de deslizamiento, verificación de reservas colaterales. | `100 - 500 $QUBIC` (~\$0.0001 - \$0.0005) | $< 0.01\text{ ms}$ ($< 10\ \mu\text{s}$) | `VOID_AB_INITIO` (Fail-Closed) |
| **Salud y Auditoría Médica** | Validación de glosas hospitalarias, consistencia de historias clínicas, ontología biológica. | `1,000 - 2,500 $QUBIC` (~\$0.001 - \$0.0025) | $< 0.05\text{ ms}$ | Rechazo causal estricto |
| **LegalTech y Auditoría de Smart Contracts** | Verificación formal de invariantes en código antes de transición de estado global. | `5,000 - 10,000 $QUBIC` (~\$0.005 - \$0.01) | $< 0.1\text{ ms}$ | Bloqueo preventivo de compilación |
| **UPoW Minero (Optimización $F \to 0$)** | Cálculo variacional y exploración de teoremas en Lean para el pool de mineros. | Subvencionado vía distribución de bloques UPoW | Tiempo real distribuido | Descarte de gradiente no convergente |

### Distribución de las Micro-tasas
* **$70\%$** Recompensas para los *Computors* que aportan capacidad de cómputo útil en el esquema UPoW de Qubic.
* **$20\%$** Fondo de Reserva y Quema (*Burn mechanism*) para conferir presión deflacionaria al ecosistema.
* **$10\%$** Mantenimiento continuo del Kernel Determinista y adición de suites de prueba formal.

---

## 2. Especificación de Endpoints REST (HTTP/2)

### Base URL:
`https://oracle.qubic-deterministic.network/v1`

### Endpoint de Verificación Principal:
`POST /v1/oracle/verify`

#### Cabeceras Requeridas:
```http
Content-Type: application/json
X-Qubic-Caller: QU_NODE_COMPUTOR_PUBLIC_KEY_HEX
X-Qubic-Epoch: 138
```

#### Ejemplo de Solicitud (Payload):
```json
{
  "vertical": "DEFI_HIGH_FREQUENCY",
  "transaction_id": "tx_qubic_8f9104c9e8314bcda901",
  "formal_expression": {
    "intent": "LIQUIDATE_POSITION",
    "collateral_ratio": "1.3540",
    "liquidation_threshold": "1.3500",
    "invariants": [
      "COLLATERAL_RATIO > LIQUIDATION_THRESHOLD",
      "ORACLE_PRICE_FEED_STALENESS < 500ms"
    ]
  },
  "max_latency_ns": 10000
}
```

#### Respuesta Determinista Exitosa (200 OK):
```json
{
  "request_id": "req_550e8400-e29b-41d4-a716-446655440000",
  "status": "STATUS_VERIFIED_SUCCESS",
  "hallucination_rate": 0.0,
  "execution_latency_us": 7.42,
  "sha256_proof_attestation": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "proof_valid": true,
  "microfee_deducted_qubic": 250,
  "timestamp": 1792497600
}
```

#### Respuesta ante Violación Axiomática (422 Unprocessable - Fail-Closed):
```json
{
  "request_id": "req_550e8400-e29b-41d4-a716-446655440001",
  "status": "STATUS_FAIL_CLOSED_VETO",
  "error_code": "VOID_AB_INITIO_AXIOM_VIOLATION",
  "message": "Causal invariant 'COLLATERAL_RATIO > LIQUIDATION_THRESHOLD' failed step-by-step verification in Kernel L0.",
  "execution_latency_us": 4.12,
  "sha256_proof_attestation": "da8f7731d10294719034981e4b901a1c97f1fbc7308d95cfcf69222c01d9f67a",
  "proof_valid": false
}
```

---

## 3. Ejemplo de Integración en C++ Nativo (Para Computors de Qubic)

```cpp
#include <iostream>
#include <chrono>
#include "qubic_deterministic_oracle.hpp"

// Interfaz ultra-rápida de enlace dinámico en silicio
int main() {
    qubic::oracle::Client oracle("127.0.0.1:50051");

    qubic::oracle::VerificationRequest req;
    req.vertical = qubic::oracle::Vertical::DEFI_HIGH_FREQUENCY;
    req.tx_hash = "qubic_tx_sample_0x7a8b9c";
    req.invariants = "assert(account_balance >= withdrawal_amount + gas_reserve)";

    auto start = std::chrono::high_resolution_clock::now();
    auto result = oracle.VerifyDeterministicProof(req);
    auto end = std::chrono::high_resolution_clock::now();

    auto duration_ns = std::chrono::duration_cast<std::chrono::nanoseconds>(end - start).count();

    if (result.is_valid()) {
        std::cout << "[✓] Atestación Formal Verificada en: " << duration_ns / 1000.0 << " us\n";
        std::cout << "[✓] Hash SHA-256: " << result.sha256_seal() << "\n";
        // Proceder con la transición de estado en el consenso
    } else {
        std::cerr << "[✗] VETO DETERMINISTA (VOID_AB_INITIO): " << result.error_message() << "\n";
        // Transacción descartada inmediatamente
    }

    return 0;
}
```

---

## 4. Garantías y Acuerdos de Nivel de Servicio (SLA)

1. **Garantía Categórica de No-Alucinación:** $\mathbf{0.000000\%}$. Ninguna transición de estado es autorizada sin una derivación formal deductiva libre de contradicciones.
2. **Latencia Sub-milisegundo en Silicio:** Media de ejecución $\le 8.5\ \mu\text{s}$, percentil 99.9 $\le 15\ \mu\text{s}$.
3. **Disponibilidad Fail-Closed:** Ante cualquier fallo de conectividad o ambigüedad semántica, el sistema responde de forma segura bloqueando la transición, previniendo estados no deterministas en la red.
