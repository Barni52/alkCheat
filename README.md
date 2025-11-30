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



    
