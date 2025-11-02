**Write css**

        using System;
        using System.IO;
        using System.Text;
        using System.Windows.Forms;
        
        public partial class MainForm : Form
        {
            public MainForm()
            {
                InitializeComponent();
            }
        
            private void btnExport_Click(object sender, EventArgs e)
            {
                SaveFileDialog saveDialog = new SaveFileDialog();
                saveDialog.Filter = "Text file (*.txt)|*.txt";
                saveDialog.Title = "Mentés másként...";
                saveDialog.FileName = "kisallatok_mentes.txt";
        
                if (saveDialog.ShowDialog() == DialogResult.OK)
                {
                    try
                    {
                        var animals = getAllAnimals(); // lekérdezés az adatbázisból
                        StringBuilder sb = new StringBuilder();
        
                        foreach (var a in animals)
                        {
                            // Feltételezve, hogy van pl. Name, Type, Age property
                            sb.AppendLine($"{a.Id};{a.Name};{a.Type};{a.Age}");
                        }
        
                        File.WriteAllText(saveDialog.FileName, sb.ToString(), Encoding.UTF8);
                        MessageBox.Show("Sikeres mentés.", "Exportálás", MessageBoxButtons.OK, MessageBoxIcon.Information);
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show("Hiba történt: " + ex.Message, "Hiba", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
        
            private List<Animal> getAllAnimals()
            {
                // Ez a te adatbázislekérdezésed helye
                // Példa dummy lista:
                return new List<Animal>
                {
                    new Animal { Id = 1, Name = "Cirmi", Type = "Macska", Age = 3 },
                    new Animal { Id = 2, Name = "Bodri", Type = "Kutya", Age = 5 }
                };
            }
        }
        
        public class Animal
        {
            public int Id { get; set; }
            public string Name { get; set; }
            public string Type { get; set; }
            public int Age { get; set; }
        }


**Read in css:**
    
    Console.WriteLine(args[0]);
    
    if (!File.Exists(args[0]))
    {
        Console.WriteLine("Wrong file path");
        Environment.Exit(1);
    }
    if (args.Length != 1)
    {
        Console.WriteLine("Wrong number of args");
        Environment.Exit(1);
    }
    
    var config = new CsvConfiguration(System.Globalization.CultureInfo.InvariantCulture)
    {
        Encoding = System.Text.Encoding.UTF8,
        Delimiter = ",",
        BadDataFound = (args) => { },
        MissingFieldFound = (args) => { },
    };
    
    using var reader = new StreamReader(args[0]);
    using var csv = new CsvReader(reader, config);
     
    var athletes = csv.GetRecords<Athlete>().ToList();

**Join:**

    var athletes = new List<Athlete>();
    var teams = new List<Team>();
    
    var result =
        from a in athletes
        join t in teams
        on a.TeamId equals t.Id
        select new { AthleteName = a.Name, TeamName = t.Name };

**Model:**

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    
    namespace ConsoleApp1.models
    {
        internal class Athlete
        {
            //ID,Name,Sex,Age,Height,Weight,Team,NOC
    
            public int ID { get; set; }
            public  String? Name { get; set; }
            public String? Sex { get; set; }
            public int? Age { get; set; }
            public int? Height { get; set; }
            public int? Weight { get; set; }
            public String? Team { get; set; }
            public String? NOC { get; set; }
    
    
        }
    }

**Type converter bullshit:**

        using System;
        using System.Collections.Generic;
        using System.ComponentModel;
        using System.Globalization;
        using System.Linq;
        using System.Text;
        using System.Threading.Tasks;
        
        namespace ConsoleApp1.models
        {
        
            public class SafeInt32Converter : Int32Converter
            {
                public override object? ConvertFrom(ITypeDescriptorContext? context, CultureInfo? culture, object value)
                {
                    if (value is string s)
                    {
                        if (int.TryParse(s, NumberStyles.Integer, culture ?? CultureInfo.InvariantCulture, out int result))
                            return result;
                        return 0;
                    }
        
                    return base.ConvertFrom(context, culture, value);
                }
        
                public override object? ConvertTo(ITypeDescriptorContext? context, CultureInfo? culture, object? value, Type destinationType)
                {
                    if (destinationType == typeof(string) && value is int i)
                        return i.ToString(culture ?? CultureInfo.InvariantCulture);
        
                    return base.ConvertTo(context, culture, value, destinationType);
                }
            }
        
        
            internal class Athlete
            {
                //ID,Name,Sex,Age,Height,Weight,Team,NOC
        
                public int ID { get; set; }
                public  string? Name { get; set; }
                public string? Sex { get; set; }
                public int? Age { get; set; }
        
                [TypeConverter(typeof(SafeInt32Converter))]
                public int? Height { get; set; }
                public int? Weight { get; set; }
                public string? Team { get; set; }
                public string? NOC { get; set; }
        
        
            }
        }


**Full code:**

    using System.Xml.Linq;
    using ConsoleApp1.models;
    using CsvHelper;
    using CsvHelper.Configuration;
    
    namespace ConsoleApp1
    {
        internal class Program
        {
            static void Main(string[] args)
            {
    
                Console.WriteLine(args[0]);
    
                if (!File.Exists(args[0]))
                {
                    Console.WriteLine("Wrong file path");
                    Environment.Exit(1);
                }
                if (args.Length != 1)
                {
                    Console.WriteLine("Wrong number of args");
                    Environment.Exit(1);
                }
    
                var config = new CsvConfiguration(System.Globalization.CultureInfo.InvariantCulture)
                {
                    Encoding = System.Text.Encoding.UTF8,
                    Delimiter = ",",
                    BadDataFound = (args) => { },
                    MissingFieldFound = (args) => { },
                };
    
                using var reader = new StreamReader(args[0]);
                using var csv = new CsvReader(reader, config);
                 
                var athletes = csv.GetRecords<Athlete>().ToList();
    
                // 3. LINQ segítségével határozd meg az összes sportoló átlagéletkorát.
    
                Console.WriteLine("3:");
                Console.WriteLine($"{athletes.Average(x => x.Age):N2}" + "\n");
    
                // 4. LINQ segítségével listázd ki a legmagasabb sportoló nevét és magasságát.
                Console.WriteLine("4:");
                int maxHeight = athletes.Max(x => x.Height);
                Athlete tallest = athletes.Where(x => x.Height == maxHeight).First();
                Console.WriteLine($"{tallest.Name}: {tallest.Height}");
    
                // 5. LINQ segítségével számold meg, hogy csapatonként hány sportoló szerepel az adathalmazban.
                Console.WriteLine("\n5:");
                var groupByTeam = athletes.GroupBy(x => x.Team).OrderByDescending(x => x.Count());
                foreach (var group in groupByTeam)
                {
                    Console.WriteLine($"{group.Key}");
                    Console.WriteLine($"{group.Count()}");
                }
    
                //6. LINQ segítségével atározd meg a női sportolók átlagmagasságát.
    
                Console.WriteLine("\n6:");
    
                Console.WriteLine(athletes.Where(x => x.Sex == "F").Average(x => x.Height));
    
                //7. LINQ segítségével listázd ki azoknak a sportolóknak a nevét ABC sorrendben, akik 25 év alattiak
                Console.WriteLine("\n7:");
    
                var youngs = athletes.Where(x => x.Age < 25).OrderBy(x => x.Name);
    
                foreach (var y in youngs)
                {
                    Console.WriteLine(y.Name);
                }
    
                //8. LINQ segítségével minden csapatból keresd meg a legnehezebb sportolót (név és testsúly)
                Console.WriteLine("\n8:");
    
                foreach (var group in groupByTeam)
                {
                    Console.WriteLine($"{group.Key} : ");
                    Console.WriteLine(group.OrderByDescending(x => x.Weight).First().Name);
                    Console.WriteLine(group.OrderByDescending(x => x.Weight).First().Weight + "\n");
                }
    
                //9. LINQ segítségével csoportosítsd a sportolókat nem szerint, és számítsd ki mindkét csoport átlagéletkorát.
                Console.WriteLine("\n9:");
    
                var groupBySex = athletes.GroupBy(x => x.Sex).OrderByDescending(x => x.Count());
    
                foreach (var group in groupBySex) 
                {
                    Console.WriteLine(group.Key + ": ");
                    Console.WriteLine(group.Average(x => x.Age));
                }
            }
        }
    }

