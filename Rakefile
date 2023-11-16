require "open3"
def capture(*args)
  o, e, s = Open3.capture3(*args)
  unless s.success?
    $stderr.puts e
    exit s.to_i
  end
  o
end

WIDTH = ENV["WIDTH"] || "95"
HEIGHT = ENV["HEIGHT"] || "95"
OFFSET_Y = (ENV["OFFSET_Y"] || "2").to_i * 0.01

srcs = Dir.glob("src/*.pdf")

srcs.each do |src|
  basename = File.basename(src, ".pdf")
  namespace src do
    desc "Convert #{src} to PNG"
    task :png do
      next if File.exists?("png/#{basename}-000.png")

      sh("convert", "-density", "600", src, "png/#{basename}-%03d.png")
    end

    desc "Crop png/#{src}-*.png"
    task :crop => :png do
      Dir.glob("png/#{basename}-???.png").each do |png|
        name = File.basename(png, ".png")
        out = "crop/#{name}-cropped.png"
        next if File.exists?(out)

        height = capture("convert", png, "-format", "%h", "info:").to_i
        sh("convert", "-gravity", "north", "-crop", "#{WIDTH}x#{HEIGHT}%+0+#{height * OFFSET_Y}", png, out)
      end
    end

    desc "Convert crop/#{src}-*-cropped.png to JPEG"
    task :jpg => :crop do
      Dir.glob("crop/#{basename}-???-cropped.png").each do |png|
        name = File.basename(png, ".png")
        out = "jpg/#{name}.jpg"
        next if File.exists?(out)

        sh("convert", png, "-background", "white", "-extent", "0x0", out)
      end
    end

    desc "Convert jpg/#{src}-*-cropped.jpg to PDF"
    task :dest => :jpg do
      out = "dest/#{basename}-cropped.pdf"
      next if File.exists?(out)

      sh("convert", *Dir.glob("jpg/#{basename}-???-cropped.jpg").to_a, out)
    end

    task :clean do
      sh "rm -f png/#{basename}-???.png"
      sh "rm -f crop/#{basename}-???-cropped.png"
      sh "rm -f jpg/#{basename}-???-cropped.jpg"
      sh "rm -f dest/#{basename}-cropped.pdf"
    end
  end
end

desc "Convert all PDFs to PNGs"
task :png => srcs.map { |src| "#{src}:png" }
desc "Convert all PDFs to PNGs and crop"
task :crop => srcs.map { |src| "#{src}:crop" }
desc "Convert all PDFs to JPG"
task :jpg => srcs.map { |src| "#{src}:jpg" }
desc "Crop all PDFs"
task :dest => srcs.map { |src| "#{src}:dest" }
desc "Clean all working files"
task :clean => srcs.map { |src| "#{src}:clean" }

task :default => :dest
