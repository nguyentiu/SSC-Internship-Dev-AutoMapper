# SSC-Internship-Dev-AutoMapper
# Hướng Dẫn Về AutoMapper trong .NET CORE
## 1. Giới Thiệu về AutoMapper
AutoMapper là một thư viện phổ biến trong .NET Core, được sử dụng để ánh xạ (mapping) giữa các đối tượng khác nhau. Trong phát triển phần mềm, đặc biệt là khi làm việc với các lớp DTO (Data Transfer Objects), việc phải viết mã để chuyển đổi giữa các đối tượng phức tạp thường chiếm nhiều thời gian và dễ gây lỗi. AutoMapper giúp tự động hóa quá trình này, giúp code gọn gàng hơn và giảm thiểu các lỗi phát sinh do việc mapping thủ công.

Lợi ích của AutoMapper:

- Tự động hóa mapping: Giúp tự động chuyển đổi giữa các đối tượng có cấu trúc tương tự.
- Giảm thiểu lỗi: Tránh các lỗi xảy ra do việc mapping thủ công.
- Tăng tốc phát triển: Giảm thời gian và công sức khi làm việc với các lớp DTO phức tạp.
## 2. Cài Đặt AutoMapper
Trước khi bắt đầu sử dụng AutoMapper, bạn cần cài đặt thư viện này thông qua NuGet Package Manager hoặc sử dụng .NET CLI:

```bash
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```
## 3. Thiết Lập AutoMapper trong .NET Core
Sau khi cài đặt xong, bạn cần cấu hình AutoMapper trong ứng dụng của mình.

### 3.1. Tạo các Profile cho Mapping

AutoMapper sử dụng các profile để xác định cách ánh xạ giữa các đối tượng. Một profile có thể được coi là một bản đồ cho các mapping.

Ví dụ, giả sử bạn có một model `Student` và một DTO `StudentDTO`, bạn có thể tạo một profile như sau:

```csharp
using AutoMapper;

public class StudentProfile : Profile
{
    public StudentProfile()
    {
        CreateMap<Student, StudentDTO>(); // Mapping từ Student sang StudentDTO
        CreateMap<StudentDTO, Student>(); // Mapping từ StudentDTO sang Student
    }
}
```
### 3.2. Đăng ký AutoMapper trong DI Container

Để AutoMapper có thể được sử dụng trong toàn bộ ứng dụng, bạn cần đăng ký nó trong `Startup.cs` hoặc `Program.cs` (trong .NET 6 trở lên):

```csharp
public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        services.AddAutoMapper(typeof(Startup)); // Đăng ký AutoMapper và scan các Profile
        // Đăng ký các service khác
    }
}
```
## 4. Sử Dụng AutoMapper trong Ứng Dụng
Khi AutoMapper đã được thiết lập, bạn có thể sử dụng nó để map giữa các đối tượng trong ứng dụng.

### 4.1. Model và DTO

Giả sử bạn có các model và DTO như sau:

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public string Grade { get; set; }
}

public class StudentDTO
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}
```
### 4.2. Sử Dụng AutoMapper trong Service hoặc Controller

Bạn có thể sử dụng AutoMapper trong service hoặc controller để chuyển đổi giữa `Student` và `StudentDTO`.

Ví dụ, sử dụng AutoMapper trong một service:

```csharp
public class StudentService : IStudentService
{
    private readonly IStudentRepository _studentRepository;
    private readonly IMapper _mapper;

    public StudentService(IStudentRepository studentRepository, IMapper mapper)
    {
        _studentRepository = studentRepository;
        _mapper = mapper;
    }

    public async Task<IEnumerable<StudentDTO>> GetAllStudentsAsync()
    {
        var students = await _studentRepository.GetAllStudentsAsync();
        return _mapper.Map<IEnumerable<StudentDTO>>(students);
    }

    public async Task<StudentDTO> GetStudentByIdAsync(int id)
    {
        var student = await _studentRepository.GetStudentByIdAsync(id);
        return _mapper.Map<StudentDTO>(student);
    }

    public async Task<int> AddStudentAsync(StudentDTO studentDto)
    {
        var student = _mapper.Map<Student>(studentDto);
        return await _studentRepository.AddStudentAsync(student);
    }

    public async Task<int> UpdateStudentAsync(StudentDTO studentDto)
    {
        var student = _mapper.Map<Student>(studentDto);
        return await _studentRepository.UpdateStudentAsync(student);
    }

    public async Task<int> DeleteStudentAsync(int id)
    {
        return await _studentRepository.DeleteStudentAsync(id);
    }
}
```
Trong đoạn mã trên:

- `IMapper` được tiêm vào service thông qua Dependency Injection.
- `_mapper.Map<TDestination>(source)` được sử dụng để chuyển đổi đối tượng từ `source` sang `destination`.
## 5. Sử Dụng AutoMapper với Các Mapping Phức Tạp
AutoMapper không chỉ đơn giản là ánh xạ giữa các thuộc tính có cùng tên, mà còn có thể xử lý các tình huống phức tạp hơn.

### 5.1. Mapping với Các Thuộc Tính Khác Tên

Nếu các thuộc tính của hai đối tượng có tên khác nhau, bạn có thể cấu hình mapping như sau:

```csharp
public class StudentProfile : Profile
{
    public StudentProfile()
    {
        CreateMap<Student, StudentDTO>()
            .ForMember(dest => dest.Name, opt => opt.MapFrom(src => src.FullName));
    }
}
```
Trong ví dụ này, `FullName` từ `Student` sẽ được map vào `Name` của `StudentDTO`.

### 5.2. Mapping với Các Đối Tượng Phức Tạp

AutoMapper cũng có thể map các đối tượng phức tạp hơn như các đối tượng chứa các collection:

```csharp
public class StudentProfile : Profile
{
    public StudentProfile()
    {
        CreateMap<Student, StudentDTO>()
            .ForMember(dest => dest.Name, opt => opt.MapFrom(src => $"{src.FirstName} {src.LastName}"));
    }
}
```
## 6. Tổng kết
AutoMapper là một công cụ mạnh mẽ và linh hoạt, giúp đơn giản hóa quá trình chuyển đổi giữa các đối tượng trong ứng dụng .NET Core. Bằng cách tự động hóa quá trình mapping, AutoMapper giúp bạn tập trung vào logic nghiệp vụ thay vì phải xử lý các chi tiết tẻ nhạt của việc chuyển đổi dữ liệu.
