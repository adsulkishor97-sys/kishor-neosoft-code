using System;
using System.Collections.Generic;
using System.Globalization;
using System.Linq;
using System.Threading.Tasks;

public class TotalCostOfOwnershipService
{
    private readonly ICurrentRepository _currentRepository;

    public TotalCostOfOwnershipService(ICurrentRepository currentRepository)
    {
        _currentRepository = currentRepository ?? throw new ArgumentNullException(nameof(currentRepository));
    }

    public async Task<GetTotalCostOfOwnershipResponse> GetTotalCostOfOwnershipByPlantIdAsync(GetTotalCostOfOwnershipRequestByplantId request)
    {
        if (request == null)
            throw new ArgumentNullException(nameof(request), "Request cannot be null");

        var (formattedStartDate, formattedEndDate, startMonth, endMonth) = FormatTcoDates(request.startDate, request.endDate);
        string plantId = request.plantId.ToString();

        var rawResponses = await GetRawTcoDataAsync(plantId, formattedStartDate, formattedEndDate, startMonth, endMonth);
        return ComputeTcoTotals(rawResponses);
    }

    // ✅ Keep date formatting isolated and simple
    private static (string formattedStartDate, string formattedEndDate, string startMonth, string endMonth)
        FormatTcoDates(string? startDate, string? endDate)
    {
        var startDateTime = ParseDateOrDefault(startDate);
        var endDateTime = ParseDateOrDefault(endDate);

        return (
            startDateTime.ToString("yyyy-MM-dd"),
            endDateTime.ToString("yyyy-MM-dd"),
            startDateTime.ToString("yyyyMM"),
            endDateTime.ToString("yyyyMM")
        );
    }

    private static DateTime ParseDateOrDefault(string? date)
    {
        return !string.IsNullOrEmpty(date)
            ? DateTime.Parse(date, CultureInfo.InvariantCulture)
            : DateTime.Now;
    }

    // ✅ Async method kept simple and parallel-safe
    private async Task<List<GetTotalCostOfOwnershipResponse>> GetRawTcoDataAsync(
        string plantId, string formattedStartDate, string formattedEndDate, string startMonth, string endMonth)
    {
        var affiliateTco = new AffiliateTcoByPlantId();
        var responseList = new List<GetTotalCostOfOwnershipResponse>();

        foreach (var item in affiliateTco.totalCostOfOwnerships)
        {
            var query = BuildTcoQuery(item.query, plantId, formattedStartDate, formattedEndDate, startMonth, endMonth);
            var results = await ExecuteTcoQueryAsync(query);
            responseList.AddRange(results);
        }

        return responseList;
    }

    private static string BuildTcoQuery(
        string query, string plantId, string formattedStartDate, string formattedEndDate, string startMonth, string endMonth)
    {
        return query
            .Replace("plantId", plantId)
            .Replace("startDate", formattedStartDate)
            .Replace("endDate", formattedEndDate)
            .Replace("formatedStartdate", startMonth)
            .Replace("formatedEnddate", endMonth);
    }

    // ✅ Executes query with minimal inline logic
    private async Task<List<GetTotalCostOfOwnershipResponse>> ExecuteTcoQueryAsync(string query)
    {
        return await _currentRepository.ExecuteBigDataQuery_New<GetTotalCostOfOwnershipResponse>(
            query,
            reader => new GetTotalCostOfOwnershipResponse
            {
                maintenance = GetDecimalValue(reader, "maintenance"),
                disposal = GetDecimalValue(reader, "disposal"),
                acquisition = GetDecimalValue(reader, "acquisition"),
                operation = GetDecimalValue(reader, "operation"),
                productionLoss = GetDecimalValue(reader, "production_loss")
            }) ?? new List<GetTotalCostOfOwnershipResponse>();
    }

    private static decimal GetDecimalValue(dynamic reader, string column)
    {
        return reader[column] != DBNull.Value ? Convert.ToDecimal(reader[column]) : 0;
    }

    // ✅ Core computation method — clean, pure, and minimal
    private static GetTotalCostOfOwnershipResponse ComputeTcoTotals(List<GetTotalCostOfOwnershipResponse> responses)
    {
        var first = responses.FirstOrDefault() ?? new GetTotalCostOfOwnershipResponse();

        var finalResponse = new GetTotalCostOfOwnershipResponse
        {
            maintenance = Math.Round(first.maintenance, 0),
            disposal = Math.Round(first.disposal, 0),
            acquisition = Math.Round(first.acquisition, 0),
            operation = Math.Round(first.operation, 0),
            productionLoss = Math.Round(first.productionLoss, 0)
        };

        finalResponse.total = finalResponse.maintenance + finalResponse.disposal +
                              finalResponse.acquisition + finalResponse.operation +
                              finalResponse.productionLoss;

        finalResponse.totalAssetOperation = finalResponse.productionLoss + finalResponse.operation + finalResponse.maintenance;

        if (finalResponse.total > 0)
            ComputeTcoPercentages(finalResponse);

        return finalResponse;
    }

    private static void ComputeTcoPercentages(GetTotalCostOfOwnershipResponse response)
    {
        response.maintenancePercent = CalculatePercentage(response.maintenance, response.total);
        response.disposalPercent = CalculatePercentage(response.disposal, response.total);
        response.acquisitionPercent = CalculatePercentage(response.acquisition, response.total);
        response.operationPercent = CalculatePercentage(response.operation, response.total);
        response.productionLossPercent = CalculatePercentage(response.productionLoss, response.total);
    }

    private static decimal CalculatePercentage(decimal value, decimal total)
    {
        return total == 0 ? 0 : Math.Round((value / total) * 100, 2);
    }
}
