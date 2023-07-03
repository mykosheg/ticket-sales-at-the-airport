# ticket-sales-at-the-airport
Development of an information and search system for ticket sales at the airport
<div class="container">
  <div class="search-wrapper">
    <div class="inputs-wrapper">
      <mat-form-field class="search-field">
        <mat-label class="search-label">Пункт відправлення</mat-label>
        <input type="text" matInput [(ngModel)]="from" class="search-input" />
      </mat-form-field>
      <mat-form-field class="search-field">
        <mat-label class="search-label">Пункт прибуття</mat-label>
        <input type="text" matInput [(ngModel)]="to" class="search-input" />
      </mat-form-field>
      <mat-form-field class="search-field">
        <mat-label class="search-label">Дата відправлення</mat-label>
        <input matInput [(ngModel)]="date" [matDatepicker]="picker" readonly />
        <mat-datepicker-toggle matSuffix [for]="picker"></mat-datepicker-toggle>
        <mat-datepicker #picker></mat-datepicker>
      </mat-form-field>
    </div>
    <button
      [disabled]="!from || !to"
      mat-raised-button
      color="primary"
      class="search-button"
      (click)="search()"
    >
      Пошук рейсів
    </button>
  </div>
  <table
    mat-table
    *ngIf="flights.length"
    [dataSource]="flights"
    class="mat-elevation-z8 flights-table"
  >
    <ng-container matColumnDef="number">
      <th mat-header-cell *matHeaderCellDef>№ рейсу</th>
      <td mat-cell *matCellDef="let element">{{ element.flight_number }}</td>
    </ng-container>

    <ng-container matColumnDef="from">
      <th mat-header-cell *matHeaderCellDef>Пункт відправлення</th>
      <td mat-cell *matCellDef="let element">
        {{ element.from }}
      </td>
    </ng-container>

    <ng-container matColumnDef="to">
      <th mat-header-cell *matHeaderCellDef>Пункт прибуття</th>
      <td mat-cell *matCellDef="let element">
        {{ element.to }}
      </td>
    </ng-container>

    <ng-container matColumnDef="fromDate">
      <th mat-header-cell *matHeaderCellDef>Дата відправлення</th>
      <td mat-cell *matCellDef="let element">
        {{ formatDate(element.start_date) }}
      </td>
    </ng-container>

    <ng-container matColumnDef="toDate">
      <th mat-header-cell *matHeaderCellDef>Дата прибуття</th>
      <td mat-cell *matCellDef="let element">
        {{ formatDate(element.end_date) }}
      </td>
    </ng-container>

    <ng-container matColumnDef="places">
      <th mat-header-cell *matHeaderCellDef>Вільних місць</th>
      <td mat-cell *matCellDef="let element">
        {{ getfreePlacesCount(element) }}
      </td>
    </ng-container>

    <ng-container matColumnDef="select">
      <th mat-header-cell *matHeaderCellDef></th>
      <td mat-cell *matCellDef="let element">
        <button
          *ngIf="!chosenFlight"
          mat-raised-button
          color="primary"
          (click)="selectFlight(element)"
        >
          Вибрати
        </button>
      </td>
    </ng-container>

    <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
    <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
  </table>
  <div *ngIf="chosenFlight" class="ticket-wrapper">
    <div class="selects-wrapper">
      <mat-form-field appearance="fill">
        <mat-label>Клас</mat-label>
        <mat-select
          [(ngModel)]="chosenClass"
          (ngModelChange)="place = null"
          name="type"
        >
          <mat-option *ngFor="let item of classes" [value]="item">
            {{ i18n.classes[item] }}
          </mat-option>
        </mat-select>
      </mat-form-field>

      <mat-form-field class="form-field">
        <mat-label class="form-label">Місце</mat-label>
        <input
          type="text"
          matInput
          [(ngModel)]="place"
          [disabled]="!chosenClass"
          class="form-input"
        />
      </mat-form-field>

      <mat-checkbox [(ngModel)]="withBaggage" [labelPosition]="'after'">
        Додатковий багаж
      </mat-checkbox>
    </div>

    <div class="sale-inputs-wrapper">
      <mat-form-field class="form-field">
        <mat-label class="form-label">Ім'я</mat-label>
        <input
          type="text"
          matInput
          [(ngModel)]="firstName"
          class="form-input"
        />
      </mat-form-field>

      <mat-form-field class="form-field">
        <mat-label class="form-label">Прізвище</mat-label>
        <input type="text" matInput [(ngModel)]="lastName" class="form-input" />
      </mat-form-field>

      <mat-form-field class="form-field">
        <mat-label class="form-label">Номер паспорту</mat-label>
        <input type="text" matInput [(ngModel)]="passport" class="form-input" />
      </mat-form-field>
    </div>
    <p *ngIf="chosenClass" class="price">Ціна білету {{ ticketPrice }} грн.</p>
    <button
      [disabled]="!(chosenClass && place && firstName && lastName && passport)"
      mat-raised-button
      color="primary"
      class="ticket-button"
      (click)="saleTicket()"
    >
      Продати
    </button>
  </div>
</div>

Код класу компонента:
import { Component, OnInit } from '@angular/core';
import { map, filter } from 'rxjs/operators';
import { ApiService } from '../api.service';
import { i18n } from '../classes.i18n';
import { Flight } from '../domain.type';

@Component({
  selector: 'app-main',
  templateUrl: './main.component.html',
  styleUrls: ['./main.component.scss'],
})
export class MainComponent implements OnInit {
  from: string = '';
  to: string = '';
  date: Date = new Date();
  i18n = i18n;

  flights: Flight[] = [];
  chosenFlight: Flight | null = null;
  chosenClass: string | null = null;
  place: number | null = null;
  withBaggage: boolean = false;
  firstName: string = '';
  lastName: string = '';
  passport: string = '';

  readonly displayedColumns = [
    'number',
    'from',
    'to',
    'fromDate',
    'toDate',
    'places',
    'select',
  ];

  constructor(private apiService: ApiService) {}

  get chosenDate() {
    return `${
      this.date.getDate() < 10 ? `0${this.date.getDate()}` : this.date.getDate()
    }.${
      this.date.getMonth() + 1 < 10
        ? `0${this.date.getMonth() + 1}`
        : this.date.getMonth() + 1
    }.${this.date.getFullYear()}`;
  }

  get classes() {
    return Object.entries(this.chosenFlight.places)
      .filter(([_, value]) => value.free_count > 0)
      .map(([key]) => key);
  }

  get ticketPrice() {
    return (
      this.chosenFlight.price[this.chosenClass] +
      (this.withBaggage ? this.chosenFlight.price.baggage : 0)
    );
  }

  getfreePlacesCount(flight: Flight) {
    return Object.entries(flight.places).reduce(
      (count, [_, value]) => count + (value.free_count ?? 0),
      0
    );
  }

  formatDate(dateStr: string) {
    const date = new Date(dateStr);
    return `${date.getDate() < 10 ? `0${date.getDate()}` : date.getDate()}.${
      date.getMonth() + 1 < 10 ? `0${date.getMonth() + 1}` : date.getMonth() + 1
    }.${date.getFullYear()} ${
      date.getHours() < 10 ? `0${date.getHours()}` : date.getHours()
    }:${date.getMinutes() < 10 ? `0${date.getMinutes()}` : date.getMinutes()}`;
  }

  clear() {
    this.chosenFlight = null;
    this.chosenClass = null;
    this.place = null;
    this.firstName = '';
    this.lastName = '';
    this.passport = '';
    this.withBaggage = false;
  }

  search() {
    this.clear();
    this.apiService
      .getFlights(this.from, this.to, this.date.toISOString())
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Flight[])
      )
      .subscribe((flights) => (this.flights = flights));
  }

  selectFlight(flight: Flight) {
    this.flights = [flight];
    this.chosenFlight = flight;
  }

  saleTicket() {
    if (
      !(
        this.chosenFlight &&
        this.chosenClass &&
        this.place &&
        this.firstName &&
        this.lastName &&
        this.passport
      )
    ) {
      return;
    }

    const { _id, flight_number } = this.chosenFlight;
    this.apiService
      .createTicket({
        flight_id: _id,
        flight_number,
        class: this.chosenClass,
        place: this.place,
        price: this.ticketPrice,
        first_name: this.firstName,
        last_name: this.lastName,
        passport: this.passport,
        baggage: this.withBaggage,
      })
      .subscribe(() => {
        this.flights = [];
        this.clear();
      });
  }

  ngOnInit(): void {}
}

Шаблон компоненту FlightsComponent:
<div class="container">
  <table
    mat-table
    *ngIf="flights.length"
    [dataSource]="flights"
    class="mat-elevation-z8 flights-table"
  >
    <ng-container matColumnDef="number">
      <th mat-header-cell *matHeaderCellDef>№ рейсу</th>
      <td mat-cell *matCellDef="let element">{{ element.flight_number }}</td>
    </ng-container>

    <ng-container matColumnDef="from">
      <th mat-header-cell *matHeaderCellDef>Пункт відправлення</th>
      <td mat-cell *matCellDef="let element">
        {{ element.from }}
      </td>
    </ng-container>

    <ng-container matColumnDef="to">
      <th mat-header-cell *matHeaderCellDef>Пункт прибуття</th>
      <td mat-cell *matCellDef="let element">
        {{ element.to }}
      </td>
    </ng-container>

    <ng-container matColumnDef="fromDate">
      <th mat-header-cell *matHeaderCellDef>Дата відправлення</th>
      <td mat-cell *matCellDef="let element">
        {{ formatDate(element.start_date) }}
      </td>
    </ng-container>

    <ng-container matColumnDef="toDate">
      <th mat-header-cell *matHeaderCellDef>Дата прибуття</th>
      <td mat-cell *matCellDef="let element">
        {{ formatDate(element.end_date) }}
      </td>
    </ng-container>

    <ng-container matColumnDef="places">
      <th mat-header-cell *matHeaderCellDef>Вільних місць</th>
      <td mat-cell *matCellDef="let element">
        {{ getfreePlacesCount(element) }}
      </td>
    </ng-container>

    <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
    <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
  </table>
  <mat-paginator
    [length]="flightCount"
    [pageSize]="limit"
    [pageSizeOptions]="[5, 10, 25, 100]"
    aria-label="Виберіть сторінку"
    (page)="onChangePage($event)"
  >
  </mat-paginator>
</div>

Код компоненту FlightsComponent:
import { Component, OnInit, ViewChild } from '@angular/core';
import { map, filter } from 'rxjs/operators';
import { MatTable } from '@angular/material/table';
import { ApiService } from '../api.service';
import { PageEvent } from '@angular/material/paginator';
import { Flight } from '../domain.type';

@Component({
  selector: 'app-flights',
  templateUrl: './flights.component.html',
  styleUrls: ['./flights.component.scss'],
})
export class FlightsComponent implements OnInit {
  flights: Flight[] = [];
  page: number = 0;
  limit: number = 5;
  flightCount: number = 100;
  displayedColumns: string[] = [
    'number',
    'from',
    'to',
    'fromDate',
    'toDate',
    'places',
  ];
  @ViewChild(MatTable, { static: false }) table!: MatTable<Flight>;
  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.apiService
      .getFlightCount()
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as number)
      )
      .subscribe((count) => (this.flightCount = count));
    this.apiService
      .findFlights(this.page * this.limit, this.limit)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Flight[])
      )
      .subscribe((flights) => (this.flights = flights));
  }

  getfreePlacesCount(flight: Flight) {
    return Object.entries(flight.places).reduce(
      (count, [_, value]) => count + (value.free_count ?? 0),
      0
    );
  }

  formatDate(dateStr: string) {
    const date = new Date(dateStr);
    return `${date.getDate() < 10 ? `0${date.getDate()}` : date.getDate()}.${
      date.getMonth() + 1 < 10 ? `0${date.getMonth() + 1}` : date.getMonth() + 1
    }.${date.getFullYear()} ${
      date.getHours() < 10 ? `0${date.getHours()}` : date.getHours()
    }:${date.getMinutes() < 10 ? `0${date.getMinutes()}` : date.getMinutes()}`;
  }

  onChangePage(event: PageEvent) {
    this.apiService
      .findFlights(event.pageIndex * event.pageSize, event.pageSize)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Flight[])
      )
      .subscribe((flights) => (this.flights = flights));
  }
}

Код сервісу ApiService:
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Flight, Ticket } from './domain.type';

type ApiResponse<T> = {
  result?: T;
  error?: {
    message: string;
  };
};

@Injectable({
  providedIn: 'root',
})
export class ApiService {
  private url = 'http://127.0.0.1:3001/api';

  constructor(private http: HttpClient) {}

  private callApi<T>(method: string, params: object = {}) {
    return this.http.post<ApiResponse<T>>(this.url, { method, params });
  }

  getFlights(
    from: string,
    to: string,
    date: string,
    offset: number = 0,
    limit: number = 10
  ) {
    return this.callApi<Flight[]>('flights.find', {
      from,
      to,
      date,
      offset,
      limit,
    });
  }

  findFlights(offset: number = 0, limit: number = 10) {
    return this.callApi<Flight[]>('flights.findAll', { offset, limit });
  }

  getFlightCount() {
    return this.callApi<number>('flights.count');
  }

  createTicket(ticket: Ticket) {
    return this.callApi<Ticket>('tickets.create', { ticket });
  }
}

Обробники запитів на серверній частині:
const FlightsRepository = require('../repositories/flights')

const flightsRepository = new FlightsRepository()

module.exports = {
  'flights.find': async ({ from, to, date, offset, limit }) => {
    const result = await flightsRepository.find(from, to, date, offset, limit)
    return { result }
  },

  'flights.findAll': async ({ offset, limit }) => {
    const result = await flightsRepository.findAll(offset, limit)
    return { result }
  },

  'flights.count': async () => {
    const result = await flightsRepository.count()
    return { result }
  },
}


const TicketsRepository = require('../repositories/tickets')
const FlightsRepository = require('../repositories/flights')

const flightsRepository = new FlightsRepository()
const ticketsRepository = new TicketsRepository()

module.exports = {
  'tickets.create': async ({ ticket }) => {
    await flightsRepository.reservePlace(ticket.flight_id, ticket.class, ticket.place)

    const result = await ticketsRepository.create(ticket)
    return { result }
  },
}

Лістинг коду репозиторіїв:
const FlightsModel = require('../models/flights')

class FlightsRepository {
  find(from, to, date, offset, limit) {
    const start = new Date(date)
    start.setHours(0, 0, 0, 0)
    const end = new Date(date)
    end.setHours(23, 59, 59, 999)
    return FlightsModel.find(
      {
        from,
        to,
        start_date: { $gte: start, $lte: end },
        $or: ['economy', 'business', 'first'].map((el) => ({
          [`places.${el}.free_count`]: { $gt: 0 },
        })),
      },
      {},
      { skip: offset, limit },
    )
  }

  findAll(offset, limit) {
    return FlightsModel.find({}, {}, { skip: offset, limit })
  }

  reservePlace(flight_id, place_class, place) {
    return FlightsModel.updateOne(
      { _id: flight_id },
      {
        $inc: { [`places.${place_class}.free_count`]: -1 },
        $push: { [`places.${place_class}.sold`]: place },
      },
    )
  }

  count() {
    return FlightsModel.countDocuments()
  }
}

module.exports = FlightsRepository

const TicketsModel = require('../models/tickets')

class TicketsRepository {
  create(ticket) {
    return TicketsModel.create({
      ...ticket,
    })
  }
}

module.exports = TicketsRepository
