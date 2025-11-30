**Video link**</br>
https://www.youtube.com/watch?v=AXnpVYU54FE



**Dependencies**

    Microsoft.EntityFrameworkCore
    Microsoft.EntityFrameworkCore.Sqlite
    Microsoft.EntityFrameworkCore.Tools

**Models**

    using System.ComponentModel.DataAnnotations.Schema;
    using System.ComponentModel.DataAnnotations;
    
    namespace WebApplication3.Models
    {
        public class Pet
        {
            public int Id { get; set; }
    
            [Required]
            [Display(Name = "Név")]
            public string Name { get; set; }
    
            [Required]
            [Range(0, 100, ErrorMessage = "Az életkor nem lehet negatív.")]
            [Display(Name = "Életkor")]
            public int Age { get; set; }
    
            [Required]
            [Range(0.0, 1000.0, ErrorMessage = "A súly nem lehet negatív.")]
            [Display(Name = "Súly (kg)")]
            public double Weight { get; set; }
    
            [Required]
            [Display(Name = "Fotó URL")]
            [RegularExpression(@".+\.(png|jpg)$",
                ErrorMessage = "Csak .png vagy .jpg kiterjesztésű URL adható meg.")]
            public string PhotoUrl { get; set; }
    
            [Required]
            [Display(Name = "Kategória")]
            public int CategoryId { get; set; }
    
            [ForeignKey(nameof(CategoryId))]
            public virtual Category? Category { get; set; }
        }
    }


---------

        using System.ComponentModel.DataAnnotations;
        
        namespace WebApplication3.Models
        {
            public class Category
            {
        
                public int Id { get; set; }
        
                [Required]
                [Display(Name = "Kategória neve")]
                public string Name { get; set; }
        
                [Required]
                [Display(Name = "Rövid leírás")]
                public string Description { get; set; }
            }
        }


**Context**

        using Microsoft.EntityFrameworkCore;
        using WebApplication3.Models;
        
        namespace WebApplication3.Context
        {
            public class EFContext : DbContext
            {
                public DbSet<WebApplication3.Models.Pet> Pets { get; set; }
                public DbSet<WebApplication3.Models.Category> Categories { get; set; }
        
                protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
                {
                    optionsBuilder.UseSqlite(@"Data Source=D:\alk2\gyakorlas\WebApplication3\Database\pets.db");
                }
        
                protected override void OnModelCreating(ModelBuilder modelBuilder)
                {
                    modelBuilder.Entity<Pet>()
                        .HasOne(p => p.Category)
                        .WithMany()
                        .HasForeignKey(p => p.CategoryId)
                        .OnDelete(DeleteBehavior.Restrict);
                }
            }
        }

-------------------------------

        namespace WebApplication3
        {
            public class Program
            {
                public static void Main(string[] args)
                {
                    var builder = WebApplication.CreateBuilder(args);
        
                    // Add services to the container.
                    builder.Services.AddControllersWithViews();
        
        
        
        
                    // EZ FONTOS
                    builder.Services.AddDbContext<Context.EFContext>();
        
        
        
        
                    var app = builder.Build();
        
                    // Configure the HTTP request pipeline.
                    if (!app.Environment.IsDevelopment())
                    {
                        app.UseExceptionHandler("/Home/Error");
                    }
                    app.UseStaticFiles();
        
                    app.UseRouting();
        
                    app.UseAuthorization();
        
                    app.MapControllerRoute(
                        name: "default",
                        pattern: "{controller=Home}/{action=Index}/{id?}");
        
                    app.Run();
                }
            }
        }


**Migration**

        Add-Migration FirstMig
        update-database


**Enums:**</br>
33:41</br>
select tag

        aps-items="Html.GeEnumSelectList<Position>()"

**Filter**

        <div>
            <form asp-action="Index" method="get">
                <div class="d-flex gap-3 align-items-center">
                    <span>
                        <label>@Html.DisplayNameFor(model => model.Position)</label>
                        <select name="positionFilter" class="form-control"
                                asp-items="Html.GetEnumSelectList<Position>()">
                            <option value=""></option>
                        </select>
                    </span>
                    <span>
                        <label>@Html.DisplayNameFor(model => model.Name)</label>
                        <input name="nameFilter" class="form-control" />
                    </span>
                    <span>
                        <label>@Html.DisplayNameFor(model => model.Retired)</label>
                        <select name="retiredFilter" class="form-control">
                            <option value=""></option>
                            <option value="true">Yes</option>
                            <option value="false">No</option>
                        </select>
                    </span>
                    <span>
                        <input type="submit" value="Filter" class="btn btn-primary" />
                    </span>
                </div>
            </form>
        </div>

------

        <th>
            @Html.DisplayNameFor(model => model.Name)
            <a asp-controller="Players" asp-action="Index" asp-route-sorting="up">&#x25B2;</a>
            <a asp-action="Index" asp-route-sorting="down">&#x25BC;</a>
        </th>

-------------------
        
        // GET: Players
        public async Task<IActionResult> Index(string? sorting, 
                                                Position? positionFilter,
                                                string? nameFilter,
                                                bool? retiredFilter)
        {
            // Ne maradjon benne a kódba!
            //Console.WriteLine(sorting + " " + nameFilter 
            //    + " " + retiredFilter + " " + positionFilter);
            var efContext = _context.Players.Include(p => p.ReferencedTeam);
        
            var query = efContext.AsQueryable();
        
            if (sorting != null)
            {
                query = sorting.Equals("up") 
                    ? query.OrderBy(e => e.Name.ToLower())
                    : query.OrderByDescending(e => e.Name.ToLower());
            }
            if (positionFilter != null)
            {
                query = query.Where(e => e.Position == positionFilter);
            }
            if (nameFilter != null)
            {
                query = query.Where(e => e.Name.ToLower().Contains(nameFilter.ToLower()));
            }
            if (retiredFilter != null)
            {
                query = query.Where(e => e.Retired == retiredFilter);
            }
        
            return View(await query.ToListAsync());
        }
