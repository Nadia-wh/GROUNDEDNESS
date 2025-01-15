

# GROUNDEDNESS




using System;
using System.Net.Http;
using System.Text.Json;
using System.Text.Json.Serialization;
using System.Threading.Tasks;


namespace data_analyses.groundednessChecker
{
    public class GroundednessDetection
    {
        private readonly HttpClient _client;
        private readonly JsonSerializerOptions _options;

        public GroundednessDetection()
        {
            _client = new HttpClient();
            _options = new JsonSerializerOptions
            {
                PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
                Converters = { new JsonStringEnumConverter() }
            };
        }

        public enum TaskType
        {
            QNA = 1
        }

        public enum DomainType
        {
            GENERIC = 1
        }

        public class QNA
        {
            public string? Query { get; set; }
        }

        public class GroundednessDetectionRequest
        {
            public string? Domain { get; set; } = DomainType.GENERIC.ToString();
            public string? Task { get; set; } = TaskType.QNA.ToString();
            public QNA? QNA { get; set; }
            public string Text { get; set; }
            public string[] GroundingSources { get; set; }
            public bool Reasoning { get; set; } = false;

            public GroundednessDetectionRequest(string text, string[] groundingSources)
            {
                if (string.IsNullOrEmpty(text))
                    throw new ArgumentException("Text cannot be null or empty.", nameof(text));

                if (groundingSources == null || groundingSources.Length == 0)
                    throw new ArgumentException("At least one grounding source must be provided.", nameof(groundingSources));

                Text = text;
                GroundingSources = groundingSources;
            }
        }

        /*  public async Task DetectGroundednessAsync(string subscriptionKey, string endpoint, string textToAnalyze, string[] groundingSources, string query)
           {
               string url = endpoint + "/contentsafety/text:detectGroundedness?api-version=2024-02-15-preview";
               _client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);

               var request = new GroundednessDetectionRequest(textToAnalyze, groundingSources)
               {
                   Task = TaskType.QNA.ToString(),
                   Domain = DomainType.GENERIC.ToString(),
                   QNA = new QNA() { Query = query },
                   Reasoning = false
               };

               string payload = JsonSerializer.Serialize(request, request.GetType(), _options);
               Console.WriteLine(payload);

               using (var content = new StringContent(payload, System.Text.Encoding.UTF8, "application/json"))
               {
                   try
                   {
                       HttpResponseMessage response = await _client.PostAsync(url, content);

                       if (response.IsSuccessStatusCode)
                       {
                           string result = await response.Content.ReadAsStringAsync();

                           Console.WriteLine("Analysis result: " + result);
                       }
                       else
                       {
                           Console.WriteLine("Error: " + response.StatusCode);
                       }
                   }
                   catch (Exception ex)
                   {
                       Console.WriteLine("Exception: " + ex.Message);
                   }
               }
           }
        */


        public async Task<string> DetectGroundednessAsync(string subscriptionKey, string endpoint, string textToAnalyze, string[] groundingSources, string query)
        {
            string url = endpoint + "/contentsafety/text:detectGroundedness?api-version=2024-02-15-preview";
            _client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);

            var request = new GroundednessDetectionRequest(textToAnalyze, groundingSources)
            {
                Task = TaskType.QNA.ToString(),
                Domain = DomainType.GENERIC.ToString(),
                QNA = new QNA() { Query = query },
                Reasoning = false
            };

            string payload = JsonSerializer.Serialize(request, request.GetType(), _options);
            Console.WriteLine(payload);

            try
            {
                using (var content = new StringContent(payload, System.Text.Encoding.UTF8, "application/json"))
                {
                    HttpResponseMessage response = await _client.PostAsync(url, content);

                    if (response.IsSuccessStatusCode)
                    {
                        string result = await response.Content.ReadAsStringAsync();
                        Console.WriteLine("Analysis result: " + result);

                        return result; // Return the analysis result
                    }
                    else
                    {
                        Console.WriteLine("Error: " + response.StatusCode);
                        return $"Error: {response.StatusCode}";
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Exception: " + ex.Message);
                return $"Exception: {ex.Message}";
            }
        }






    }





}
