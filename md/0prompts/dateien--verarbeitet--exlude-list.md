# EXCLUDE-LIST – bereits verarbeitete Dateien

Stand: 2026-03-12 | Session: yam2 Coaching

Folgende Dateien sind vollständig restrukturiert und bereinigt.
**Nicht erneut anfassen** – außer bei gezieltem Bug-Fix.

---

## Core/

- [x] `AsyncRelayCommand.cs`
- [x] `EnumExtensions.cs`
- [x] `ObservableObject.cs`
- [x] `ObservableValidator.cs`
- [x] `RelayCommand.cs`
- [x] `SqlExtensions.cs`

## Models/

- [x] `Member.cs` (v3.4 – ZahlungsartId entfernt, DSGVO-Flags, Clone/CopyFrom)
- [x] `Fee.cs` (RemainingAmount OnPropertyChanged ergänzt)
- [x] `Payment.cs`
- [x] `User.cs` (UserIsAuth entfernt)
- [x] `LookupItem.cs`
- [x] `MemberEnums.cs` (Ausgetreten=6 ergänzt)
- [x] `Enums.cs` (Anrede-Duplikat entfernt, nur Zahlungsart)

## Services/ (Interfaces)

- [x] `IMemberRepository.cs` (DeleteAsync-Kommentar auf Soft-Delete)
- [x] `IFinanceRepository.cs`
- [x] `IUserService.cs`
- [x] `IViewService.cs` (Confirm() ergänzt)

## Services/ (Implementierungen)

- [x] `SqlRepositoryBase.cs`
- [x] `SqlMemberRepository.cs` (Soft-Delete, WithConnectionAsync, lokales _connectionString entfernt)
- [x] `SqlFinanceRepository.cs` (catch-Fix, lokales _connectionString entfernt)
- [x] `UserService.cs` (UserIsAuth entfernt)
- [x] `ViewService.cs` (Confirm() implementiert)

## ViewModels/

- [x] `MembersViewModel.cs` (AsyncRelayCommand, IsLoading-Fixes, IViewService.Confirm)
- [x] `MainViewModel.cs` (unnötige usings entfernt)
- [x] `MemberDetailViewModel.cs` (public-Modifier ergänzt)
- [x] `FeeDetailViewModel.cs` (ASSISTANT-Marker entfernt)
- [x] `PaymentDetailViewModel.cs` (ASSISTANT-Marker entfernt, default(DateTime)-Fix)

---

## Noch offen

- [ ] `HomeViewModel.cs`
- [ ] `App.xaml.cs` (F3: Verbindungsstring)
- [ ] `Views/*.xaml`
