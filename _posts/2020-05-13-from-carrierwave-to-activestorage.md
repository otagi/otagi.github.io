---
title: Migrating from CarrierWave to Active Storage
---

The story of how we moved all our uploaded files from [CarrierWave](https://github.com/carrierwaveuploader/carrierwave) to [Active Storage](https://guides.rubyonrails.org/active_storage_overview.html) in our big monolithic [Ruby on Rails](https://www.rubyonrails.org/) app.

## The legacy

Our application at [Spreds](https://www.spreds.com/) first used CarrierWave for uploads. It was a nice solution at the time, handling the basics of storing files and resizing images.

Managing disk space wasn’t something we wanted to worry about so we switched to cloud-based file storage (Google Cloud Storage). This added [Fog](https://github.com/fog/fog) and [CarrierWave Direct](https://github.com/dwilkie/carrierwave_direct) to the mix.

It worked well. Except when we had the occasional upload bug. Hours were then spent tracing back the code path through all the involved gems: CarrierWave, CarrierWave Direct, Fog, Fog-code, Fog-google, Fog-xml, and others. Our upload stack became a behemoth of unnecessary complexity for such a trivial task.

When Active Storage was announced, I was impressed by its clean API, and immediately wanted to switch. The catch is that we had to find a way to move the existing files to the new storage structure without breaking the app.

## Storage structure

Our CarrierWave uploads use a naming scheme that looks like this:

```
/uploads/<model_name>/<attribute_name>/<model_id>/<file_name>
```

with the additional twist that CarrierWave Direct replaces the `model_id` with a UUID, and stores it alongside the filename in the database column.

Active Storage uses a less human-friendly structure:

```
/<file_key>
```

where `file_key` is a UUID.

My first idea was to simply rename the actual files on the bucket with `gsutil` (the Google Cloud CLI), and generate the corresponding `ActiveStorage::Attachment` and `ActiveStorage::Blob` records. Easy peasy.

After some attempts, that path proved to be too bug-prone. Cloud storage services tend to be very picky about the right combination of ACL, IAM, and other obscure file permission initialisms. Plus, the Active Storage public API is minimalist on purpose, has its own way of fetching metadata like the content type or the image size, and I wasn’t an expert of its inner workings.

The slower but safer way is to download each file and re-upload it through the official Active Storage method. Less hidden surprises, and still an option since our website’s file collection is big but not (yet) on a YouTube-like level.

## Playing it safe

The conversion process:

1. Replace the `mount_uploader :photo, PhotoUploader` in the model with `has_one_attached :photo`.
1. Replace the associated code in controllers and views with an Active Storage-compatible version.
1. Delete the `PhotoUploader` class.
1. Deploy the modified code.
1. Run our custom `active_storage:migrate[ModelClass,photo]` task to copy the actual files to the new structure.
1. Remove the `photo` column from the database table.
1. Manually delete the original files from the bucket.

The goal is to be able to convert one attribute at a time. Old not-yet-converted CarrierWave uploads will continue to work alongside the new converted Active Storage ones. It is much easier and safer to check that the conversion didn’t break anything when taking small steps.

## How it played out

Our app had a lot of these uploader attributes. We opened one GitHub Pull Request per attribute. The whole operation was spread over several months, progressing in parallel with other development work.

Since it’s a multistep, semi-automated process, we sometimes forgot to clean up old columns and files. But that didn’t impact the end product, and was easily fixed.

Access to some files was broken during the few seconds or minutes between the deploy and the task completion, but we deemed that acceptable.

The short iteration cycle also meant development mistakes were easier to spot and rectify. And we didn’t have to shut down operations to move the whole shebang in one go. The safe and slow method proved successful, and went smoothly, all things considered.

## The rake task

```ruby
require 'ansi_string' # Custom lib to colorize the console output. Not essential.
require 'fog_helper'  # Custom thin wrapper lib around Fog calls, to avoid authentication boilerplate.
require 'open-uri'

namespace :active_storage do
  desc "Attach old CarrierWave files to existing ActiveStorage attachment"
  task :migrate, [:model_name, :attachment_name] => :environment do |_, args|
    Model = args[:model_name].classify.constantize
    uploader_name = attachment_name = args[:attachment_name].to_sym

    model_scope = Model.all
    model_scope = model_scope.where.not(uploader_name => [nil, ''])

    model_scope.find_each do |model_instance|
      # We don't have the uploader accessor anymore, but can still read the column value.
      carrierwave_file_name = model_instance.read_attribute(uploader_name)

      # Useful sanity check, just in case.
      if carrierwave_file_name.blank?
        puts "- ##{model_instance.id}: " + "no carrierwave filename".yellow
        next
      end

      # Uniformize differences between CarrierWave and CarrierWave Direct filename conventions.
      carrierwave_file_name.prepend("#{model_instance.id}/") unless carrierwave_file_name.include?('/')

      attachment     = model_instance.send(attachment_name)
      file_name      = File.basename(carrierwave_file_name)
      file_extension = File.extname(file_name)[1..-1]
      file_path      = "uploads/#{Model.name.underscore}/#{uploader_name}/#{carrierwave_file_name}"
      file_url       = FogHelper.file(file_path)&.url(10.minutes.from_now) # use Fog to get a valid URL to the actual file
      file_mime_type = Mime::Type.lookup_by_extension(file_extension).to_s

      # Sanity check, in case the actual file went AWOL.
      if file_url.blank?
        puts "- ##{model_instance.id}: #{carrierwave_file_name} " + "no carrierwave file".yellow
        next
      end

      # Sanity check, in case the attachment has already been converted.
      if attachment.attached?
        puts "- ##{model_instance.id}: #{file_name} " + "already attached".yellow
        next
      end

      # Copy the file, and attach it to the model.
      begin
        file = open(file_url)
        attachment.attach(io: file, filename: file_name, content_type: file_mime_type)
        puts "- ##{model_instance.id}: #{file_name} " + (attachment.attached? ? "attached".green : "not attached".red)
      rescue OpenURI::HTTPError
        puts "- ##{model_instance.id}: #{file_name} " + "could not open carrierwave file".red
      end
    end
  end
end
```

## Aknowledgements

Thanks to Marie Hargitt for her help on running this lengthy migration process.

Thanks to Simon Schoeters for his support during the development, and the editing help on this article.

