### Laravel Media Library
এই ডকুমেন্টে Spatie-র laravel-medialibrary প্যাকেজ নিয়ে বিস্তারিত আলোচনা করা হয়েছে: এটি কী, কেন ব্যবহার করা হয়, ভেতরে কীভাবে কাজ করে, 
কোড উদাহরণ, সুবিধা-অসুবিধা এবং best practice।
### Laravel Media Library কী?
Laravel Media Library (সাধারণভাবে Spatie Media Library) হলো Spatie কোম্পানির তৈরি একটি জনপ্রিয় ওপেন-সোর্স Laravel প্যাকেজ (spatie/laravel-medialibrary)। এর মূল কাজ হলো ইমেজ,
PDF, ভিডিও, ডকুমেন্ট ইত্যাদি ফাইলকে সরাসরি Eloquent মডেলের সাথে যুক্ত (associate) করা এবং সেগুলো সহজে সংরক্ষণ, ম্যানেজ ও প্রদর্শন করা।

__সহজ ভাষায়:__ User মডেলের একটি প্রোফাইল ছবি আছে, Post মডেলের আছে ফিচার্ড ইমেজ ও গ্যালারি, Product মডেলের আছে একাধিক ছবি ও ম্যানুয়াল PDF। 
এই সবকিছুর জন্য আলাদা আলাদা কলাম, ফোল্ডার ও আপলোড লজিক লেখার বদলে এই প্যাকেজ একটি একীভূত ও পরিচ্ছন্ন সমাধান দেয়।
#
### মূল বৈশিষ্ট্যে 
- যেকোনো Eloquent মডেলে এক বা একাধিক ফাইল সংযুক্ত করা
- ফাইলকে Collection (যেমন avatar, gallery, documents) অনুযায়ী ভাগ করা
- ইমেজ থেকে স্বয়ংক্রিয়ভাবে Conversion তৈরি (thumbnail, resize, crop, format পরিবর্তন)
- Responsive images (srcset সহ)
- Laravel-এর Filesystem disks (local, public, S3, DigitalOcean Spaces ইত্যাদি) সাপোর্ট
- ফাইলের সাথে custom properties (alt text, caption, ইত্যাদি) সংরক্ষণ
- Queue ব্যবহার করে ব্যাকগ্রাউন্ডে conversion তৈরি
- URL, request, স্ট্রিং বা স্ট্রিম থেকে ফাইল যোগ করা
- মডেল ডিলিট হলে সংশ্লিষ্ট ফাইলও স্বয়ংক্রিয়ভাবে মুছে ফেলা
### কেন ব্যবহার করা হয়?
__প্যাকেজ ছাড়া সাধারণ সমস্যা :__

প্যাকেজ ছাড়া ফাইল আপলোড হ্যান্ডেল করতে গেলে সাধারণত এসব ঝামেলা হয়:

- প্রতিটি টেবিলে image, avatar, file_path ধরনের আলাদা কলাম যোগ করতে হয়।
- একাধিক ছবির জন্য আলাদা টেবিল ও রিলেশন বানাতে হয়।
- থাম্বনেইল বা বিভিন্ন সাইজ তৈরির জন্য নিজেকে Intervention Image ইত্যাদি দিয়ে কোড লিখতে হয়।
- পুরনো ফাইল মুছে ফেলার লজিক ভুলে গেলে সার্ভারে অব্যবহৃত ফাইল জমে যায়।
- স্টোরেজ লোকাল থেকে S3-তে সরাতে গেলে অনেক কোড বদলাতে হয়।
- ফাইলের নাম ও পাথের কোনো নির্দিষ্ট কাঠামো থাকে না।

__এই প্যাকেজ কোন সমস্যা সমাধান করে :__

| সমস্যা | Media Library-র সমাধান |
|---|---|
| প্রতি মডেলে আলাদা কলাম | একটি কেন্দ্রীয় media টেবিল (polymorphic) |
| একাধিক ফাইল ম্যানেজ | Collection সিস্টেম |
| থাম্বনেইল/রিসাইজ | registerMediaConversions() |
| ডিস্ক পরিবর্তন | disk কনফিগ বদলালেই হয় |
| ফাইল ক্লিনআপ | মডেল ডিলিট হলে মিডিয়াও ডিলিট|
| Metadata | custom_properties JSON কলাম |

__কোন কোন Project এর জন্য উপযোগী:__

- ই-কমার্স (প্রোডাক্টের গ্যালারি, ভেরিয়েন্ট ইমেজ)
- ব্লগ ও CMS (ফিচার্ড ইমেজ, অ্যাটাচমেন্ট)
- ইউজার প্রোফাইল (অ্যাভাটার, কভার ফটো)
- ডকুমেন্ট ম্যানেজমেন্ট (PDF, Word ফাইল)
- অ্যাডমিন প্যানেল (Filament, Nova ইত্যাদির সাথে ইন্টিগ্রেশন)
### ইনস্টলেশন ও সেটআপ
__প্রয়োজনীয়তা:__ সর্বশেষ সংস্করণের জন্য নির্দিষ্ট PHP ও Laravel ভার্সন লাগে। ইনস্টলের আগে অফিসিয়াল ডকুমেন্টেশনে সামঞ্জস্য ভার্সন দেখে নেওয়া ভালো। 
ইমেজ প্রসেসিংয়ের জন্য সার্ভারে GD অথবা Imagick এক্সটেনশন থাকতে হবে।
- ধাপ ১: প্যাকেজ ইনস্টল
```
composer require spatie/laravel-medialibrary
```
- ধাপ ২: মাইগ্রেশন পাবলিশ ও রান
```
php artisan vendor:publish --provider="Spatie\MediaLibrary\MediaLibraryServiceProvider" --tag="medialibrary-migrations"
php artisan migrate
```
এতে media নামে একটি টেবিল তৈরি হবে।
- ধাপ ৩: কনফিগ ফাইল পাবলিশ (ঐচ্ছিক)
```
php artisan vendor:publish --provider="Spatie\MediaLibrary\MediaLibraryServiceProvider" --tag="medialibrary-config"
```
এরপর config/media-library.php ফাইলে disk, max file size, queue, image driver ইত্যাদি কনফিগার করা যায়।
- ধাপ ৪: Storage লিংক
```
php artisan storage:link
```
- ৪. মডেল প্রস্তুত করা
যে মডেলে ফাইল যুক্ত করতে চান, সেখানে HasMedia ইন্টারফেস ইমপ্লিমেন্ট করুন এবং InteractsWithMedia ট্রেইট ব্যবহার করুন।
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Spatie\MediaLibrary\HasMedia;
use Spatie\MediaLibrary\InteractsWithMedia;
use Spatie\MediaLibrary\MediaCollections\Models\Media;

class Post extends Model implements HasMedia
{
    use InteractsWithMedia;

    // Collection নির্ধারণ
    public function registerMediaCollections(): void
    {
        // একটি মাত্র ফাইল রাখবে (নতুন দিলে পুরনোটা মুছে যাবে)
        $this->addMediaCollection('cover')
            ->singleFile();

        // শুধু নির্দিষ্ট ধরনের ফাইল গ্রহণ করবে
        $this->addMediaCollection('gallery')
            ->acceptsMimeTypes(['image/jpeg', 'image/png', 'image/webp']);

        // ফাইল না থাকলে ডিফল্ট URL
        $this->addMediaCollection('avatar')
            ->useFallbackUrl('/images/default-avatar.png');
    }

    // Conversion নির্ধারণ
    public function registerMediaConversions(?Media $media = null): void
    {
        $this->addMediaConversion('thumb')
            ->width(368)
            ->height(232)
            ->sharpen(10)
            ->performOnCollections('gallery', 'cover');
    }
}
```

### ব্যবহারের উদাহরণ
__ফাইল যোগ করা__
```
// ফাইল পাথ থেকে
$post->addMedia($pathToFile)->toMediaCollection('gallery');

// রিকোয়েস্ট থেকে (সবচেয়ে প্রচলিত)
$post->addMediaFromRequest('image')->toMediaCollection('cover');

// একাধিক ফাইল
$post->addMultipleMediaFromRequest(['images'])
    ->each(fn ($file) => $file->toMediaCollection('gallery'));

// URL থেকে
$post->addMediaFromUrl('https://example.com/photo.jpg')->toMediaCollection('gallery');

// Custom property সহ
$post->addMedia($file)
    ->withCustomProperties(['alt' => 'সমুদ্রের ছবি', 'caption' => 'কক্সবাজার'])
    ->toMediaCollection('gallery');
```
৫.২ ফাইল পড়া ও URL নেওয়া
// প্রথম ফাইলের URL
$url = $post->getFirstMediaUrl('cover');

// Conversion সহ URL
$thumbUrl = $post->getFirstMediaUrl('gallery', 'thumb');

// Collection-এর সব মিডিয়া
$images = $post->getMedia('gallery');

foreach ($images as $media) {
    echo $media->getUrl();         // মূল ফাইল
    echo $media->getUrl('thumb');  // thumb conversion
    echo $media->file_name;
    echo $media->size;             // বাইটে
    echo $media->getCustomProperty('alt');
}

__Blade-এ ব্যবহার__
```
<img src="{{ $post->getFirstMediaUrl('cover', 'thumb') }}" alt="{{ $post->title }}">

{{-- Responsive image --}}
{{ $post->getFirstMedia('cover') }}
```
__ডিলিট ও ক্লিয়ার__
```
// নির্দিষ্ট মিডিয়া মুছুন
$post->getFirstMedia('gallery')->delete();

// পুরো collection খালি করুন
$post->clearMediaCollection('gallery');
```
__Controller-এ সম্পূর্ণ উদাহরণ__
```
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|string',
        'cover' => 'required|image|max:2048',
        'gallery.*' => 'image|max:4096',
    ]);

    $post = Post::create($request->only('title'));

    $post->addMediaFromRequest('cover')->toMediaCollection('cover');

    if ($request->hasFile('gallery')) {
        $post->addMultipleMediaFromRequest(['gallery'])
            ->each(fn ($f) => $f->toMediaCollection('gallery'));
    }

    return redirect()->route('posts.show', $post);
}
```
### ভিতরে কীভাবে কাজ করে?
__ডাটাবেস স্ট্রাকচার:__

সব ফাইলের তথ্য একটি কেন্দ্রীয় media টেবিলে জমা থাকে। টেবিলটি polymorphic relation ব্যবহার করে, ফলে একটি টেবিলই সব মডেলের ফাইল ধারণ করতে পারে।
| কলাম | কাজ |
|---|---|
| model_type, model_id | কোন মডেলের কোন রেকর্ডের ফাইল |
| uuid | ইউনিক আইডেন্টিফায়ার |
| collection_name | ফাইল কোন collection-এর |
| name, file_name | ফাইলের নাম |
| mime_type, size | ধরন ও আকার |
| disk, conversions_disk | কোথায় সংরক্ষিত |
| manipulations | ইমেজ ম্যানিপুলেশন তথ্য (JSON) |
| custom_properties | কাস্টম মেটাডাটা (JSON) |
| generated_conversions | কোন conversion তৈরি হয়েছে |
| responsive_images | responsive ইমেজের তথ্য |
| order_column | সাজানোর ক্রম |

__ফাইল আপলোডের প্রবাহ:__

- আপনি addMedia() বা addMediaFromRequest() কল করেন।
- প্যাকেজ ফাইলটি যাচাই করে (collection-এর MIME ও নিয়ম অনুযায়ী)।
- media টেবিলে একটি রেকর্ড তৈরি হয়।
- Path Generator ঠিক করে ফাইলটি ডিস্কের কোন ফোল্ডারে যাবে (ডিফল্টে মিডিয়ার id অনুযায়ী ফোল্ডার)।
- ফাইলটি কনফিগার করা disk-এ কপি/মুভ হয়।
- মডেলে সংজ্ঞায়িত conversion থাকলে সেগুলো তৈরির জন্য job ডিসপ্যাচ হয় (ডিফল্টে queue-তে)।
- Conversion তৈরি হলে generated_conversions আপডেট হয়।
  
__ডিস্কে ফাইলের কাঠামো (ডিফল্ট):__
```
storage/app/public/
└── 1/                  <- media id
    ├── photo.jpg       <- মূল ফাইল
    ├── conversions/
    │   └── photo-thumb.jpg
    └── responsive-images/
        └── ...
```
__Conversion প্রক্রিয়া:__

প্যাকেজ নিজে ইমেজ প্রসেস করে না। এটি ভেতরে spatie/image ব্যবহার করে, যা GD বা Imagick ড্রাইভারের ওপর নির্ভরশীল। 
width(), height(), crop(), sharpen(), format(), quality() ইত্যাদি মেথড দিয়ে আপনি ইমেজ রূপান্তর নির্ধারণ করেন। 
ভারী ইমেজের জন্য conversion queue-তে চালানোই ভালো, যাতে ইউজারকে অপেক্ষা করতে না হয়। প্রয়োজনে ->nonQueued() দিয়ে সঙ্গে সঙ্গে তৈরি করা যায়।
```
$this->addMediaConversion('card')
    ->width(600)
    ->format('webp')
    ->quality(80)
    ->withResponsiveImages()
    ->queued();
```
__ইভেন্ট ও স্বয়ংক্রিয় ক্লিনআপ:__
মডেল ডিলিট হলে প্যাকেজ তার সংযুক্ত সব মিডিয়া ও ফাইল মুছে দেয় (সফট ডিলিট ব্যবহার করলে ফাইল থেকে যায়, কারণ মডেল আসলে ডিলিট হয়নি)।
এজন্য আলাদা কোড লিখতে হয় না।
### গুরুত্বপূর্ণ ফিচারের বিস্তারিত
__Collections:__

Collection হলো একটি মডেলের ফাইলগুলোকে যৌক্তিক ভাগে ভাগ করার উপায়। প্রতিটি collection-এর আলাদা নিয়ম থাকতে পারে:

- singleFile(): শুধু একটি ফাইল
- onlyKeepLatest(3): সর্বশেষ ৩টি রাখবে
- acceptsMimeTypes([...]): নির্দিষ্ট MIME
- acceptsFile(fn ($file) => ...): কাস্টম শর্ত
- useDisk('s3'): collection-ভিত্তিক আলাদা ডিস্ক
- useFallbackUrl(): ফাইল না থাকলে বিকল্প URL
  
__Responsive Images:__
withResponsiveImages() ব্যবহার করলে প্যাকেজ একই ছবির একাধিক সাইজ তৈরি করে এবং srcset সহ <img> ট্যাগ জেনারেট করে। 
ফলে মোবাইল ডিভাইসে ছোট ছবি লোড হয় এবং পেজের গতি বাড়ে।

__Custom Properties:__
প্রতিটি মিডিয়ার সাথে JSON আকারে যেকোনো অতিরিক্ত তথ্য রাখা যায়:
```
$media->setCustomProperty('alt', 'ছবির বর্ণনা');
$media->save();

$media->getCustomProperty('alt');
```
__Temporary URL (S3 ইত্যাদি):__
প্রাইভেট ফাইলের জন্য সীমিত সময়ের URL তৈরি করা যায়:
```
$url = $media->getTemporaryUrl(now()->addMinutes(10));
```
__Artisan কমান্ড:__
conversion আবার তৈরি করতে
```
php artisan media-library:regenerate
```
অব্যবহৃত ফাইল ও অসঙ্গতি পরিষ্কার করতে 
```
php artisan media-library:clean
```
__ডাউনলোড ও জিপ:__

একটি মিডিয়াকে সরাসরি ডাউনলোড রেসপন্স হিসেবে ফেরত দেওয়া যায়:
return $media; // Responsable, সরাসরি ডাউনলোড হবে
একাধিক মিডিয়া একটি ZIP ফাইল হিসেবেও স্ট্রিম করা সম্ভব।

__কনফিগারেশনের গুরুত্বপূর্ণ অপশন:__

| অপশন | বিবরণ |
|---|---|
| disk_name | ডিফল্ট ডিস্ক |
| max_file_size | সর্বোচ্চ ফাইল সাইজ |
| queue_connection_name ও queue_name | conversion job-এর queue |
| queue_conversions_by_default | ডিফল্টে queue ব্যবহার হবে কি না |
| image_driver | gd বা imagick |
| path_generator | ফাইলের পাথ তৈরির ক্লাস |
| url_generator | URL তৈরির ক্লাস |
| media_model | কাস্টম Media মডেল |

### সুবিধা (Advantages)
1. কোড কমে ও পরিচ্ছন্ন হয়: আপলোড, পাথ, ডিলিট, রিসাইজ সব একটি জায়গা থেকে পরিচালিত হয়।
2. একাধিক ফাইল সহজে ম্যানেজ: আলাদা টেবিল ও রিলেশন বানাতে হয় না, Collection দিয়েই হয়ে যায়।
3. স্বয়ংক্রিয় image conversion: থাম্বনেইল, ওয়েবপি রূপান্তর, রিসাইজ সহজ।
4. Responsive images: পারফরম্যান্স ও SEO উন্নত হয়।
5. ডিস্ক-নিরপেক্ষ: local থেকে S3-তে যাওয়া মূলত কনফিগারেশনের বিষয়।
6. Polymorphic ডিজাইন: যেকোনো মডেলে ব্যবহার করা যায়।
7. স্বয়ংক্রিয় ক্লিনআপ: মডেল ডিলিট হলে ফাইলও মুছে যায়।
8. Queue সাপোর্ট: ভারী প্রসেসিং ব্যাকগ্রাউন্ডে চলে।
9. বড় কমিউনিটি ও নিয়মিত আপডেট: Spatie-র প্যাকেজ হওয়ায় ভালো ডকুমেন্টেশন ও মেইনটেন্যান্স আছে।
10. ইকোসিস্টেম ইন্টিগ্রেশন: Filament, Nova, Livewire ইত্যাদির সাথে প্লাগইন/সাপোর্ট পাওয়া যায়।
11. এক্সটেনসিবল: Path Generator, URL Generator, Media model, Conversion সবই কাস্টমাইজ করা যায়।
12. নিরাপত্তা সহায়ক: MIME ও collection-ভিত্তিক ফিল্টারিং এবং temporary URL সুবিধা আছে।
###  অসুবিধা (Disadvantages)
1. শেখার বক্ররেখা: Collection, Conversion, Path Generator, Queue ইত্যাদি বুঝতে নতুনদের কিছুটা সময় লাগে।
2. ছোট প্রকল্পে অতিরিক্ত ভারী: শুধু একটি অ্যাভাটার আপলোড করার জন্য এটি overkill হতে পারে।
3. অতিরিক্ত ডাটাবেস কোয়েরি: মিডিয়া লোড করলে N+1 সমস্যা হতে পারে। with('media') দিয়ে eager load করতে হয়।
4. Conversion-এর জন্য সার্ভার রিসোর্স: GD/Imagick ও queue worker প্রয়োজন, বড় ছবিতে মেমোরি বেশি লাগে।
5. Queue সেটআপ না থাকলে বিভ্রান্তি: Queue worker না চালালে conversion তৈরি হয় না, যা নতুনদের প্রায়ই সমস্যায় ফেলে।
6. মাইগ্রেশন জটিলতা: আগে থেকে চালু প্রকল্পে পুরনো ফাইল এই সিস্টেমে আনতে কাস্টম স্ক্রিপ্ট লাগে।
7. ডিফল্ট ফোল্ডার কাঠামো: আইডি-ভিত্তিক ফোল্ডারে ফাইলের নাম থেকে বিষয়বস্তু বোঝা কঠিন, বদলাতে হলে কাস্টম Path Generator লিখতে হয়।
8. প্যাকেজ-নির্ভরতা (vendor lock-in): ভবিষ্যতে অন্য সিস্টেমে যেতে চাইলে media টেবিলের ডেটা রূপান্তর করতে হবে।
9. ফ্রন্টএন্ড আপলোডার নেই (ফ্রি ভার্সনে): ড্র্যাগ-অ্যান্ড-ড্রপ UI বা প্রিভিউয়ের জন্য আলাদা লাইব্রেরি বা পেইড Media Library Pro লাগে।
10. ভার্সন সামঞ্জস্যতা: নতুন মেজর ভার্সনে PHP/Laravel-এর নির্দিষ্ট ভার্সন লাগে এবং আপগ্রেডে breaking change থাকতে পারে।
11. ভিডিও প্রসেসিং সীমিত: ভিডিও থেকে থাম্বনেইল বা ট্রান্সকোডিংয়ের জন্য অতিরিক্ত টুল (যেমন FFmpeg সম্পর্কিত সমাধান) লাগে।
### কখন ব্যবহার করা উচিত, কখন ব্যবহার করা উচিত না:
| ব্যবহার করুন যখন | ব্যবহার না করলেও চলে যখন |
|---|---|
| একাধিক মডেলে ফাইল আছে | শুধু ১-২টি ফাইল সহ একটি ছোট অ্যাপ |
| থাম্বনেইল/রিসাইজ দরকার | কোনো ইমেজ প্রসেসিং দরকার নেই |
| S3 বা ক্লাউড স্টোরেজ ব্যবহার করবেন | শুধু সাধারণ Storage::put দিয়ে কাজ চলে |
| দীর্ঘমেয়াদি, বড় প্রকল্প | দ্রুত প্রোটোটাইপ |
| ফাইলের metadata দরকার | শুধু পাথ সংরক্ষণই যথেষ্ট |
### বিকল্প সমাধান

| বিকল্প | বৈশিষ্ট্য |
|---|---|
| Laravel Storage (নেটিভ) | সহজ, কিন্তু সব লজিক নিজে লিখতে হয় |
| Intervention Image | শুধু ইমেজ ম্যানিপুলেশন, মডেল সংযুক্তি নেই |
| Plank Mediable | মডেলে মিডিয়া সংযুক্ত করার আরেকটি প্যাকেজ |
| Cloudinary / ImageKit | ক্লাউড-ভিত্তিক ইমেজ সার্ভিস, অন-দ্য-ফ্লাই ট্রান্সফর্ম |
| Filament-এর বিল্ট-ইন আপলোড | অ্যাডমিন প্যানেলের জন্য সুবিধাজনক |

### Best Practices

1. সবসময় Collection ব্যবহার করা এবং অর্থবহ নাম দেয়া (avatar, gallery)।
2. একক ফাইলের ক্ষেত্রে singleFile() দেয়া, যাতে পুরনো ফাইল জমে না থাকে।
3. Validation অবশ্যই করুন: image, mimes, max নিয়ম দিন। acceptsMimeTypes() শুধু প্যাকেজ স্তরের অতিরিক্ত সুরক্ষা।
4. Queue worker চালু রাখুন (Supervisor দিয়ে) প্রোডাকশনে।
5. লিস্ট পেজে with('media') দিয়ে eager load করা।
6. বড় ছবি আপলোড হলে conversion ব্যবহার করে ছোট সংস্করণ দেখান, মূল ফাইল নয়।
7. প্রাইভেট ফাইল private disk ও temporary URL দিয়ে রক্ষা করা।
8. webp ফরম্যাটে conversion করলে ফাইল সাইজ উল্লেখযোগ্যভাবে কমে।
9. Alt text ও caption custom_properties-এ রাখা (accessibility ও SEO-র জন্য)।
10. কনফিগে max_file_size সীমিত রাখা এবং সার্ভারের upload_max_filesize/post_max_size-ও মিলিয়ে দেখা।
11. ডিপ্লয়মেন্টে php artisan storage:link চালাতে হবে।
12. Conversion-এর সংজ্ঞা বদলালে media-library:regenerate চালানো।
### সাধারণ সমস্যা ও সমাধান

| সমস্যা | সম্ভাব্য কারণ ও সমাধান |
|---|---|
| ছবি দেখা যাচ্ছে না (404) | storage:link চালানো হয়নি, বা APP_URL ভুল |
| Conversion তৈরি হচ্ছে না | Queue worker চলছে না; php artisan queue:work চালানো বা ->nonQueued() ব্যবহার করতে হবে |
| "GD/Imagick not found" | সার্ভারে সংশ্লিষ্ট PHP এক্সটেনশন ইনস্টল করুন |
| বড় ফাইল আপলোড ব্যর্থ | php.ini-তে upload_max_filesize ও post_max_size বাড়াতে হবে |
| পাতা ধীর | N+1 সমস্যা, eager loading করুন |
| S3 তে ফাইল যাচ্ছে না | ডিস্ক কনফিগ ও credentials পরীক্ষা করতে হবে |

### সারসংক্ষেপ
Spatie Laravel Media Library হলো Laravel অ্যাপ্লিকেশনে ফাইল ম্যানেজমেন্টের জন্য একটি পরিণত, নমনীয় ও বহুল ব্যবহৃত সমাধান। এটি ফাইলকে Eloquent মডেলের সাথে সুন্দরভাবে যুক্ত করে, ইমেজ conversion ও responsive image স্বয়ংক্রিয় করে এবং বিভিন্ন স্টোরেজ ডিস্কের সাথে সহজে কাজ করে।
মাঝারি থেকে বড় প্রকল্পে, যেখানে একাধিক মডেলে ছবি বা ডকুমেন্ট আছে এবং থাম্বনেইল বা ক্লাউড স্টোরেজ দরকার, সেখানে এটি সময় ও কোড দুটোই বাঁচায়। অন্যদিকে খুব ছোট প্রকল্পে বা শুধু একটি সাধারণ আপলোডের জন্য এটি অতিরিক্ত জটিলতা আনতে পারে, সেক্ষেত্রে Laravel-এর নেটিভ Storage-ই যথেষ্ট।
### রেফারেন্স
- অফিসিয়াল ডকুমেন্টেশন: spatie.be/docs/laravel-medialibrary
- GitHub রিপোজিটরি: github.com/spatie/laravel-medialibrary
- Media Library Pro (পেইড UI কম্পোনেন্ট): medialibrary.pro
নোট: ভার্সনভেদে মেথডের নাম ও কনফিগ সামান্য ভিন্ন হতে পারে। ইনস্টলের আগে নিজের Laravel ও PHP ভার্সনের জন্য অফিসিয়াল ডকুমেন্টেশন দেখে নেয়া উচিত।
