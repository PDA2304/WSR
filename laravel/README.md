Создание миграции контройлера и модели php artisan make:model Название --resource -m

Пример request

public function rules()
    {
        return [
            "surname" => ["required","min:5","max:30"],
            "name" => ["required","min:3","max:30"],
            "middle_name" => ["max:30"],
            "salary" => ["required","numeric","min:0","not_in:0"],
            "passport_data" => ["required","numeric","digits:10","not_in:0",new ExistsEmployeeData],
            "post_id" => ["required","numeric",function ($attribute, $value, $fail) {
                if (!Post::where('id','=',$value)->first()) {
                    $fail('Такой должности нет');
                }
            }],
        ];
    }


Пример модели
protected $table = "purchase";

    protected $fillable =[
        'id','purchase_date', 'receipt_date', 'purchase_total','employee_id','provider_id'
    ];

    public function purchase_ingredients()
    {
        return $this->hasMany("App\Models\PurchaseIngredients");
    }

    public function employee()
    {
        return $this->belongsTo("App\Models\Employees");
    }


    public function provider()
    {
        return  $this->belongsTo("App\Models\Provider");
    }

    protected $hidden = [
        'updated_at',
        'created_at',
    ];


Пример Resource

public function toArray($request)
    {
        return [
            "id"=> $this->id,
            "purchase_date"=>$this->purchase_date,
            "receipt_date"=> $this->receipt_date,
            "purchase_total"=> $this->ipurchase_totald,
            "employee_id"=> $this->employee,
            "provider_id"=> $this->provider,
            "purchase_ingredients"=>  $this->purchase_ingredients
        ];
    }
Пример миграции 

Schema::create('purchase_ingredients', function (Blueprint $table) {
            $table->id();
            $table->integer('purchase_id')->unsigned();
            $table->integer('purchase_ingredients_cost_unit')->unsigned();
            $table->integer('purchase_ingredients_total')->unsigned();
            $table->integer('ingredients_id')->unsigned();
            $table->date('medical_start_date')->nullable()->useCurrent = true;
            $table->timestamps();


            $table->foreign('purchase_id')->references('id')->on('purchase')->onDelete('cascade');
            $table->foreign('ingredients_id')->references('id')->on('ingredients')->onDelete('cascade');
        });


Пример контройлера

<?php

namespace App\Http\Controllers;

use App\Http\Requests\PurchaseIngredientsRequest;
use App\Http\Resources\PurchaseIngredientsResource;
use App\Http\Resources\PurchaseResource;
use App\Models\Purchase;
use App\Models\PurchaseIngredients;
use Illuminate\Http\Request;

class PurchaseIngredientsController extends Controller
{
    /**
     *
     */
    public function index()
    {
        $result  = PurchaseIngredientsResource::collection(PurchaseIngredients::orderBy("id")->get());
        return  response()->json($result,200);
    }

    /**
     *
     */
    public function store(PurchaseIngredientsRequest $request)
    {
        $result = PurchaseIngredients::create($request->all());
        if(PurchaseIngredients::where("purchase_ingredients_id" ,"=", $this->purchase_ingredients_id)->where("ingredients_id" ,"=", $this->ingredients_id)->first())
        {
            response()->json(["ingredients_id" => ["Такой продукт уже был добавлен в закупочный лист"]],200);
        }
        return response()->json(new PurchaseIngredientsResource($result),200);
    }

    /**
     *
     */
    public function show($id)
    {
        $result = PurchaseIngredients::find($id);
        if($result == null) return response()->json(["message"=> "Такого продукта нет"],404);

        return response()->json(new PurchaseIngredientsResource($result),200);
    }

    /**
     *
     */
    public function update(PurchaseIngredientsRequest $request,$id)
    {
        $result = PurchaseIngredients::find($id);
        if($result == null) return response()->json(["message"=> "Такого продукта нет"],404);
        if($request->purchase_ingredients_id != $result->purchase_ingredients_id)
        {
            if(PurchaseIngredients::where("purchase_ingredients_id" ,"=", $this->purchase_ingredients_id)->where("ingredients_id" ,"=", $this->ingredients_id)->first())
            {
                response()->json(["ingredients_id" => ["Такой продукт уже был добавлен в закупочный лист"]],200);
            }
        }
        $result->update($request->all());
        return response()-> json(new PurchaseResource($result),200);
    }

    /**
     *
     */
    public function destroy($id)
    {
        $result = PurchaseIngredients::find($id);
        if($result == null) return response()->json(["message"=> "Такого продукта нет"],404);
        return response()->json(["message"=>"Продукт был удален из листа"]);
    }
}

