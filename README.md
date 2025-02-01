# CPF Validation Project With Serverless Azure Microservice

In this project, we show the steps to create a serverless microservice for CPF validation using Azure.

## 📄 Estrutura do Código

The main code of the microservice was written in .NET and is shown below:

```csharp
using System;
using System.IO;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace httpValidaCpf
{
    public static class fnvalidacpf
    {
        [FunctionName("fnvalidacpf")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            if (data == null || string.IsNullOrEmpty(data?.cpf?.ToString()))
            {
                return new BadRequestObjectResult("Por favor, informe o CPF.");
            }

            string cpf = data?.cpf;
            log.logInformation(cpf);

            if (ValidaCPF(cpf, log) == false)
            {
                return new BadRequestObjectResult("CPF inválido.");
            }

            var responseMessage = "CPF válido, e não consta na base de dados de fraudes, e não consta na base de dados de débitos.";

            return new OkObjectResult(responseMessage);
        }

        public static bool ValidaCPF(string cpf)
        {
            if (string.IsNullOrEmpty(cpf))
                return false;

            cpf = cpf.Trim().Replace(".", "").Replace("-", "");

            if (cpf.Length != 11)
                return false;

            int[] multiplicador1 = new int[9] { 10, 9, 8, 7, 6, 5, 4, 3, 2 };
            int[] multiplicador2 = new int[10] { 11, 10, 9, 8, 7, 6, 5, 4, 3, 2 };

            string tempCpf = cpf.Substring(0, 9);
            int soma = 0;

            for (int i = 0; i < 9; i++)
                soma += int.Parse(tempCpf[i].ToString()) * multiplicador1[i];

            int resto = soma % 11;
            if (resto < 2)
                resto = 0;
            else
                resto = 11 - resto;

            string digito = resto.ToString();
            tempCpf += digito;
            soma = 0;

            for (int i = 0; i < 10; i++)
                soma += int.Parse(tempCpf[i].ToString()) * multiplicador2[i];

            resto = soma % 11;
            if (resto < 2)
                resto = 0;
            else
                resto = 11 - resto;

            digito += resto.ToString();

            return cpf.EndsWith(digito);
        }
    }
}


```


Remembering that there is a need to edit the local.settings.json file like this:


```json

{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnetIsolated"
  }
}

```

## 🛠 Requisitos
Antes de iniciar, certifique-se de ter instalado:
- **.NET SDK 6.0+**
- **Azure Functions Core Tools**
- **Postman ou cURL** para testar a API
