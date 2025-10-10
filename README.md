public async Task<GetTotalCostOfOwnershipResponse> GetTotalCostOfOwnershipByPlantIdAsync(GetTotalCostOfOwnershipRequestByplantId request)
{

    var plantId = request?.plantId;
    //here get  json file data           
    var startDateTime = request?.startDate != null ? DateTime.Parse(request.startDate, CultureInfo.InvariantCulture) : DateTime.Now;
    var endDateTime = request?.endDate != null ? DateTime.Parse(request.endDate, CultureInfo.InvariantCulture) : DateTime.Now;
    //convert format as yyyy-mm-dd 2025-08-01
    var formatedStartdate = startDateTime.ToString("yyyy-MM-dd");
    var formatedEnddate = endDateTime.ToString("yyyy-MM-dd");
    //convert format as yyyymm 202508
    var startDateMonth = startDateTime.ToString("yyyyMM");
    var endDateMonth = endDateTime.ToString("yyyyMM");
    List<GetTotalCostOfOwnershipResponse> response = new List<GetTotalCostOfOwnershipResponse>();
    AffiliateTcoByPlantId affiliateTcoByPlantId = new AffiliateTcoByPlantId();
    foreach (var item in affiliateTcoByPlantId.totalCostOfOwnerships)
    {
        var query = item.query!
           .Replace("plantId", plantId.ToString())
            .Replace("startDate", formatedStartdate)
            .Replace("endDate", formatedEnddate)
            .Replace("formatedStartdate", startDateMonth)
            .Replace("formatedEnddate", endDateMonth);



        response = await _currentRepository.ExecuteBigDataQuery_New<GetTotalCostOfOwnershipResponse>(query, reader => new GetTotalCostOfOwnershipResponse
        {
            maintenance = reader["maintenance"] != DBNull.Value ? Convert.ToDecimal(reader["maintenance"]) : 0,
            disposal = reader["disposal"] != DBNull.Value ? Convert.ToDecimal(reader["disposal"]) : 0,
            acquisition = reader["acquisition"] != DBNull.Value ? Convert.ToDecimal(reader["acquisition"]) : 0,
            operation = reader["operation"] != DBNull.Value ? Convert.ToDecimal(reader["operation"]) : 0,
            productionLoss = reader["production_loss"] != DBNull.Value ? Convert.ToDecimal(reader["production_loss"]) : 0,



        }) ?? new List<GetTotalCostOfOwnershipResponse>();
    }
    var finalResponse = new GetTotalCostOfOwnershipResponse
    {
        maintenance = Math.Round(response.FirstOrDefault()?.maintenance ?? 0, 0),
        disposal = Math.Round(response.FirstOrDefault()?.disposal ?? 0, 0),
        acquisition = Math.Round(response.FirstOrDefault()?.acquisition ?? 0, 0),
        operation = Math.Round(response.FirstOrDefault()?.operation ?? 0, 0),
        productionLoss = Math.Round(response.FirstOrDefault()?.productionLoss ?? 0, 0)



    };

    finalResponse.total = finalResponse.maintenance + finalResponse.disposal + finalResponse.acquisition + finalResponse.operation + finalResponse.productionLoss;
    finalResponse.totalAssetOperation = finalResponse.productionLoss + finalResponse.operation + finalResponse.maintenance;


    if (finalResponse.total > 0)
    {
        finalResponse.maintenancePercent = Math.Round((finalResponse.maintenance / finalResponse.total) * 100, 2);
        finalResponse.disposalPercent = Math.Round((finalResponse.disposal / finalResponse.total) * 100, 2);
        finalResponse.acquisitionPercent = Math.Round((finalResponse.acquisition / finalResponse.total) * 100, 2);
        finalResponse.operationPercent = Math.Round((finalResponse.operation / finalResponse.total) * 100, 2);
        finalResponse.productionLossPercent = Math.Round((finalResponse.productionLoss / finalResponse.total) * 100, 2);
    }
    return finalResponse;
}
