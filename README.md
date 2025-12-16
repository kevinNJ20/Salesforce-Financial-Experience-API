# Salesforce Financial Experience API

API Experience pour les Customer Service Representatives (CSR) dans Salesforce Financial Services Cloud.

## Description

Cette API fournit une vue financière détaillée pour les agents du service client dans Salesforce. Elle permet de récupérer le résumé financier d'un client à partir de son ID Salesforce.

## Endpoints

### GET /api/salesforce/financial/customers/{customerGlobalId}
Récupère le résumé financier CSR pour un client identifié par son Global ID.

**Note:** Le paramètre URI est `customerGlobalId` mais dans le contexte Salesforce, cela représente l'ID Salesforce (sfId). Le flow extrait ensuite le Global ID depuis les external IDs du client Salesforce.

**Response:** CSRFinancialSummary object avec:
- Informations client (Salesforce ID, Global ID)
- Liste des comptes
- Liste des cartes

## Configuration

### Connexions HTTP Requises

Cette API fait des appels HTTP vers:
- **Salesforce Customers System API** (port 8081) - pour récupérer le client et extraire le Global ID
- **Bank Accounts Process API** (port 8082) - pour récupérer le résumé financier

Configurer dans `global.xml`:
- `Salesforce_Customers_System_API_Config`
- `Bank_Accounts_Process_API_Config`

### Port

- **Port HTTP**: 8083

## Architecture Technique

### Flows Business-Logic

- `get-csr-financial-summary-business-logic`: 
  1. Récupère le client depuis Salesforce via son ID
  2. Extrait le Global ID depuis les external IDs
  3. Si Global ID trouvé, récupère le résumé financier depuis Bank Accounts Process API
  4. Sinon, retourne une réponse indiquant que le client n'est pas encore synchronisé

### Logique de Résolution d'ID

Le flow gère deux identifiants:
- **Salesforce ID (sfId)**: ID du Contact dans Salesforce (utilisé pour l'appel initial)
- **Global ID**: ID global du client dans Global Data (extraite depuis externalIds)

## Exemples de Requêtes

### GET /api/salesforce/financial/customers/{customerGlobalId}

```bash
curl -X GET "http://localhost:8083/api/salesforce/financial/customers/003xx000004TmiQAAS"
```

**Response (si Global ID trouvé):**
```json
{
  "customer": {
    "salesforceId": "003xx000004TmiQAAS",
    "globalId": "550e8400-e29b-41d4-a716-446655440000"
  },
  "accounts": [...],
  "creditCards": [...]
}
```

**Response (si Global ID non trouvé):**
```json
{
  "customer": {
    "salesforceId": "003xx000004TmiQAAS",
    "globalId": null
  },
  "accounts": [],
  "creditCards": [],
  "message": "Customer not yet synchronized to Global Data"
}
```

