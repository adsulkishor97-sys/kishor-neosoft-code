using Xunit;
using Moq;
using AutoFixture;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;

public class AdminControllerTests
{
    private readonly IFixture _fixture;
    private readonly Mock<IAdminServices> _adminServiceMock;
    private readonly AdminController _controller;

    public AdminControllerTests()
    {
        _fixture = new Fixture();
        _adminServiceMock = new Mock<IAdminServices>();
        _controller = new AdminController(_adminServiceMock.Object);
    }

    [Fact]
    public async Task GetUserActivityDataWithDynamicSearch_ShouldReturnOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetLoggingDataWithDynamicSearchRequest>();
        var response = _fixture.Build<GetActivityTrackerLogsRepositoryLayerResponse>()
                               .With(x => x.data, _fixture.CreateMany<GetActivityTrackerLogsStoredProcedureResponse>(3).ToList())
                               .Create();

        _adminServiceMock
            .Setup(s => s.GetActivityTrackerLogsAsync(request.pageNumber, request.pageSize, request.keyword!))
            .ReturnsAsync(response);

        // Act
        var result = await _controller.GetUserActivityDataWithDynamicSearch(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var returnedData = Assert.IsType<GetActivityTrackerLogsRepositoryLayerResponse>(okResult.Value);

        Assert.Equal(request.pageNumber, returnedData.pageNumber);
        Assert.Equal(request.pageSize, returnedData.pageSize);
        Assert.NotEmpty(returnedData.data!);
        _adminServiceMock.Verify(s => s.GetActivityTrackerLogsAsync(request.pageNumber, request.pageSize, request.keyword!), Times.Once);
    }

    [Fact]
    public async Task GetUserActivityDataWithDynamicSearch_ShouldReturnNoContent_WhenDataIsNull()
    {
        // Arrange
        var request = _fixture.Create<GetLoggingDataWithDynamicSearchRequest>();
        GetActivityTrackerLogsRepositoryLayerResponse? response = null;

        _adminServiceMock
            .Setup(s => s.GetActivityTrackerLogsAsync(request.pageNumber, request.pageSize, request.keyword!))
            .ReturnsAsync(response);

        // Act
        var result = await _controller.GetUserActivityDataWithDynamicSearch(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
        _adminServiceMock.Verify(s => s.GetActivityTrackerLogsAsync(request.pageNumber, request.pageSize, request.keyword!), Times.Once);
    }

    [Fact]
    public async Task GetUserActivityDataWithDynamicSearch_ShouldReturnNoContent_WhenDataIsEmpty()
    {
        // Arrange
        var request = _fixture.Create<GetLoggingDataWithDynamicSearchRequest>();
        var response = _fixture.Build<GetActivityTrackerLogsRepositoryLayerResponse>()
                               .With(x => x.data, new List<GetActivityTrackerLogsStoredProcedureResponse>())
                               .Create();

        _adminServiceMock
            .Setup(s => s.GetActivityTrackerLogsAsync(request.pageNumber, request.pageSize, request.keyword!))
            .ReturnsAsync(response);

        // Act
        var result = await _controller.GetUserActivityDataWithDynamicSearch(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
        _adminServiceMock.Verify(s => s.GetActivityTrackerLogsAsync(request.pageNumber, request.pageSize, request.keyword!), Times.Once);
    }
}

// ✅ Mock interface (must match your real one)
public interface IAdminServices
{
    Task<GetActivityTrackerLogsRepositoryLayerResponse> GetActivityTrackerLogsAsync(int pageNumber, int pageSize, string keyword);
}

// ✅ Dummy controller (replace namespace/class name with yours)
public class AdminController : ControllerBase
{
    private readonly IAdminServices _adminServices;

    public AdminController(IAdminServices adminServices)
    {
        _adminServices = adminServices;
    }

    public async Task<IActionResult> GetUserActivityDataWithDynamicSearch(GetLoggingDataWithDynamicSearchRequest request)
    {
        var result = await _adminServices.GetActivityTrackerLogsAsync(request.pageNumber, request.pageSize, request.keyword!);
        if (result == null || result.data == null || result.data.Count == 0)
        {
            return NoContent();
        }

        result.pageSize = request.pageSize;
        result.pageNumber = request.pageNumber;
        return Ok(result);
    }
}
