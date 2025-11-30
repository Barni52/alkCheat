**Video link**\n
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
