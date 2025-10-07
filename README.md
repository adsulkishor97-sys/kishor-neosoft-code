public async Task<IActionResult> GetTotalCostOfOwnershipByPlantId(GetTotalCostOfOwnershipRequestByplantId request)
{
    var totalCostOfOwnershipResult = await _currentServices.GetTotalCostOfOwnershipByPlantIdAsync(request);
    if (totalCostOfOwnershipResult! == null) return NoContent();
    else return Ok(totalCostOfOwnershipResult);
}
public class GetTotalCostOfOwnershipRequestByplantId
{
    public int plantId { get; set; }
    public string? startDate { get; set; }
    public string? endDate { get; set; }
    public int? affiliateId { get; set; }
}
public class GetTotalCostOfOwnershipResponse
{
    public decimal maintenance { get; set; }
    public decimal disposal { get; set; }
    public decimal acquisition { get; set; }
    public decimal operation { get; set; }
    public decimal productionLoss { get; set; }
    public decimal total { get; set; }
    public decimal maintenancePercent { get; set; }
    public decimal disposalPercent { get; set; }
    public decimal acquisitionPercent { get; set; }
    public decimal operationPercent { get; set; }
    public decimal productionLossPercent { get; set; }
    public decimal totalAssetOperation { get; set; }


}
Task<GetTotalCostOfOwnershipResponse> GetTotalCostOfOwnershipByPlantIdAsync(GetTotalCostOfOwnershipRequestByplantId request);
